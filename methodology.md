# Methodology

How each benchmark in this repository was measured.

---

## Data Collection

All metrics are derived from production call sessions stored in PostgreSQL (Supabase). No synthetic data, no test calls, no simulated environments. Every number in this repository comes from real outbound BPO calls + inbound callbacks to real phone numbers during February 2026 – May 2026.

Call metadata (duration, outcome, channel, timestamps, transfer flags) is written at call completion by a post-call processing worker. Latency measurements are captured server-side during active calls.

**Exclusions from this dataset:**
- Demo / test phone lines
- EU-region subclients on the reseller plane (separate pool, not statistically representative yet)
- Internal QA calls
- Calls with missing `ended_at` timestamp (i.e., never properly closed — typically zero)

---

## Hardware

| Component | Specification |
|-----------|--------------|
| SIP Bridge Server (US) | GCP e2-standard-2 (2 vCPU, 8GB RAM), us-central1 (Council Bluffs, Iowa) |
| SIP Bridge Server (EU) | GCP e2-standard-2, europe-west1 |
| OS | Debian 12 |
| AI Workers | Cloudflare Workers (edge compute) |
| Database | PostgreSQL via Supabase (US-East-1 + EU-Central-1) |
| Bridge runtime | Go 1.25, single binary, systemd-managed |

The SIP bridge server handles SIP registration, RTP audio processing, WebSocket bridging, and bridge-side TEN VAD barge-in detection. The AI pipeline (STT, LLM, TTS) runs on third-party inference APIs, not on the VM itself.

---

## AI Stack

| Component | Provider | Model | Protocol |
|-----------|----------|-------|----------|
| Speech-to-Text | Deepgram | Flux v2 (`flux-general-en`) | Streaming WebSocket, 16kHz linear16 |
| Language Model (primary) | Google | Gemini 2.5 Flash | Streaming HTTP, tool calling enabled, temperature 0.6 |
| Language Model (alternative) | Groq | Llama 3.1 8B Instant | Streaming HTTP, temperature 0.6, no tool calling |
| Text-to-Speech | Cartesia | Sonic-3 | Persistent WebSocket, native mulaw 8kHz |
| Barge-in (Layer 2, bridge-side) | TEN-framework | TEN VAD | C library on bridge VM, on raw µ-law RTP |
| Barge-in (Layer 1, worker-side) | Deepgram | Flux VAD | Endpointing within STT WebSocket |

All three primary pipeline components (STT/LLM/TTS) use streaming protocols — the LLM begins generating before STT is fully complete, and TTS begins synthesizing before the full LLM response is available. This pipelining is critical to achieving sub-500ms end-to-end latency.

**LLM choice by client:**
Paying clients with transfer/qualification flows that benefit from structured tool calls (`transfer_to_specialist`, `hangup_call`) run Gemini 2.5 Flash. Clients on the lower-cost / simpler-flow path run Groq Llama 3.1 8B. The benchmarks above mix calls across both LLM paths.

---

## Metric Definitions

### Response Latency

**Definition:** Time from end of caller's speech (as determined by STT turn detection) to first audio byte of AI response sent via RTP.

**How measured:** Server-side timestamps at two points:
1. STT `EndOfTurn` event received
2. First TTS audio chunk written to RTP stream

**What it includes:** STT finalization, LLM inference (first token), TTS synthesis (first audio frame), and internal processing overhead.

**What it excludes:** Network transit time from caller's phone to our server and back. Codec encoding/decoding time at the endpoints. Jitter buffer delays at the caller's device.

**Conditions:**
- "Direct SIP" = AI server registered as SIP extension on VICIdial Asterisk. RTP flows directly between Asterisk and our server.
- "Twilio relay" = Audio routes through Twilio's Media Streams WebSocket, adding relay overhead.
- "HTTP API" = Industry estimate for platforms that integrate via synchronous HTTP calls mid-dialplan. Not directly measured by us.

### Barge-In Latency

**Definition:** Time from caller starting to speak (during AI playback) to AI audio playback being stopped.

**Layer 2 (bridge-side TEN VAD):** TEN VAD analyzes raw µ-law RTP frames on the bridge VM. Detects voiced speech in ~30ms. Sends `caller_speech_start` WebSocket event to worker, which drains the outbound audio channel.

**Layer 1 (worker-side Deepgram VAD fallback):** If TEN VAD does not fire first, Deepgram's VAD endpointing within the STT WebSocket eventually triggers (~200-300ms). Worker stops AI audio playback.

First-wins semantics — whichever layer detects first wins.

### Call Duration

**Definition:** Wall-clock time from call answer to call termination.

- For SIP calls: From 200 OK response to SIP INVITE, to BYE or server-initiated disconnect.
- For Twilio calls: From Twilio `call.status = in-progress` to call completion.

**Measured by:** Server-side timestamps recorded in the call session. Not derived from telephony billing records.

### Transfer Rate

**Definition:** Number of calls where the AI initiated a warm transfer to a human agent, divided by total calls in that category.

**What counts as a transfer:** The AI must have:
1. Conducted a qualification conversation
2. Determined the caller meets transfer criteria
3. Initiated the transfer action (VICIdial `ra_call_control&stage=EXTENSIONTRANSFER` admin API call, OR SIP REFER, OR Twilio Dial redirect)
4. The transfer reached SUCCESS status (200 OK from the dialer side)

Calls where the caller asked to speak to a human but did not meet qualification criteria are not counted as transfers — they are categorized as `info_provided`, `complaint_logged`, or `other`.

**Denominator:** Total calls includes all call attempts — answered, unanswered, busy, voicemail, and failed. This is a conservative calculation. When measured against only answered-and-engaged calls, effective transfer rates are higher.

### Voicemail Detection

**Detection method:** Two-layer approach:
1. **Carrier-level AMD (Answering Machine Detection):** Runs asynchronously at the telephony layer. Analyzes initial audio patterns (greeting length, cadence) to determine if a human or machine answered.
2. **Keyword backup:** If AMD does not trigger, the AI listens for voicemail indicator phrases ("leave a message," "not available," "beep," etc.) and disconnects.

**Detection speed:** Measured from call answer to disconnect decision. Median 7.7 seconds, average 11.5 seconds. The gap between median and average reflects the long tail of edge cases (long voicemail greetings that AMD missed and keyword backup catches later).

**Note on BYOD/SIP calls:** For calls arriving via VICIdial's dialer, the dialer often performs its own AMD before connecting to our AI. The 6.0% voicemail rate in our data represents voicemails that passed through the dialer's detection — it is not the raw voicemail rate of the underlying phone lists.

---

## Anonymization

This dataset contains only aggregate metrics. The following information is explicitly excluded:

- Client names, company names, or campaign identifiers
- Phone numbers (caller or callee)
- Caller names or any personally identifiable information
- Transcript content
- Geographic information beyond server region
- Individual call records

Vertical names (SSDI, Debt Relief, ACA, Final Expense, Medicare, Health Insurance) describe the type of campaign, not specific clients. Multiple distinct clients contributed to several verticals (e.g., SSDI numbers aggregate across multiple SSDI campaign deployments).

---

## Limitations

1. **Latency is server-side only.** End-to-end latency experienced by the caller includes network transit, jitter buffers, and codec delays that we do not measure. Actual perceived latency is likely 50-150ms higher than our reported numbers.

2. **AMD false positive rate is unknown.** When AMD incorrectly classifies a human as a voicemail (or vice versa), we have no direct way to measure this from our side. The detection happens at the carrier or dialer level before reaching our AI.

3. **Transfer rates depend heavily on list quality.** The same AI agent on a high-quality lead list will show higher transfer rates than on a cold list. Our benchmarks reflect the lists used by our clients during this period — your results will vary.

4. **Varying sample sizes across verticals.** SSDI (19,200 calls) and Final Expense Callbacks (5,791) + ACA (6,287) provide strong statistical confidence. Medicare (608) and inbound Final Expense (24) are small samples — interpret with caution.

5. **Newer verticals reflect ramp-up dynamics.** Some sub-samples (e.g. a 315-call SSDI sample over 4 days and a 608-call Medicare sample over 1 day) reflect very recent campaign starts where list quality, prompt tuning, and operational state had not yet stabilized. Their transfer rates should be interpreted as early-ramp data, not steady-state.

6. **3.5-month window.** Call center performance varies by day of week, time of day, season, and regulatory environment. A longer measurement period would provide more stable benchmarks. We extended from the original 2-month window for this release.

7. **Single geographic region for the dataset.** The benchmarked calls run through the US bridge (us-central1). EU calls are excluded from this dataset because that traffic is not yet at production scale.

8. **LLM path is mixed.** Some calls use Gemini 2.5 Flash (with tool calling), others use Groq Llama 3.1 8B (without tool calling). Aggregate transfer rates may obscure provider-specific differences. We have not yet broken down transfer rate by LLM provider.

---

## Reproducibility

These benchmarks reflect a specific configuration:

- **STT:** Deepgram Flux v2 with `eot_threshold=0.7`
- **LLM (primary):** Gemini 2.5 Flash via Google AI Studio (streaming, temperature 0.6, tool calling enabled)
- **LLM (alternative):** Llama 3.1 8B on Groq inference (streaming, temperature 0.6)
- **TTS:** Cartesia Sonic-3 with persistent WebSocket and native mulaw output
- **Telephony:** Direct SIP registration on Asterisk 16.x (chan_sip)
- **Server:** GCP e2-standard-2 in us-central1
- **Barge-in:** TEN VAD (bridge-side, Apache 2.0 release from TEN-framework) + Deepgram VAD (worker-side fallback)

Changing any of these components — different STT provider, different LLM size, different TTS engine, different server location — will produce different results. These benchmarks are published as a reference point, not as guarantees.

---

## Updates

This repository will be updated as more data is collected. Each update will note the new measurement period and any changes to the AI stack or infrastructure.

**Release history:**
- **2026-03**: Initial release — 5,939 calls across 3 verticals (SSDI, Final Expense, ACA), Llama 3.1 8B on Groq + Cartesia Sonic Turbo
- **2026-05** (this release): 35,865 calls across 6 verticals, added Gemini 2.5 Flash with tool calling, Cartesia Sonic-3, bridge-side TEN VAD Layer 2 barge-in

Last updated: May 2026
