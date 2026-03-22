# Methodology

How each benchmark in this repository was measured.

---

## Data Collection

All metrics are derived from production call sessions stored in PostgreSQL (Supabase). No synthetic data, no test calls, no simulated environments. Every number in this repository comes from real outbound BPO calls to real phone numbers during February–March 2026.

Call metadata (duration, outcome, channel, timestamps) is written at call completion by a post-call processing worker. Latency measurements are captured server-side during active calls.

---

## Hardware

| Component | Specification |
|-----------|--------------|
| SIP Bridge Server | GCP e2-standard-2 (2 vCPU, 8GB RAM) |
| Region | us-central1 (Council Bluffs, Iowa) |
| OS | Ubuntu 22.04 LTS |
| AI Workers | Cloudflare Workers (edge compute) |
| Database | PostgreSQL via Supabase (US-East-1) |

The SIP bridge server handles SIP registration, RTP audio processing, and WebSocket bridging. The AI pipeline (STT, LLM, TTS) runs on third-party inference APIs, not on the VM itself.

---

## AI Stack

| Component | Provider | Model | Protocol |
|-----------|----------|-------|----------|
| Speech-to-Text | Deepgram | Flux v2 (`flux-general-en`) | Streaming WebSocket, 16kHz linear16 |
| Language Model | Groq | Llama 3.1 8B Instant | Streaming HTTP, temperature 0.6 |
| Text-to-Speech | Cartesia | Sonic Turbo | Persistent WebSocket, native mulaw 8kHz |

All three components use streaming protocols — the LLM begins generating before STT is fully complete, and TTS begins synthesizing before the full LLM response is available. This pipelining is critical to achieving sub-500ms end-to-end latency.

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
3. Initiated the transfer action (SIP REFER or Twilio redirect)

Calls where the caller asked to speak to a human but did not meet qualification criteria are not counted as transfers — they are categorized as "info provided" or "other."

**Denominator:** Total calls includes all call attempts — answered, unanswered, busy, voicemail, and failed. This is a conservative calculation. When measured against only answered-and-engaged calls, effective transfer rates are higher.

### Voicemail Detection

**Detection method:** Two-layer approach:
1. **Carrier-level AMD (Answering Machine Detection):** Runs asynchronously at the telephony layer. Analyzes initial audio patterns (greeting length, cadence) to determine if a human or machine answered.
2. **Keyword backup:** If AMD does not trigger, the AI listens for voicemail indicator phrases ("leave a message," "not available," "beep," etc.) and disconnects.

**Detection speed:** Measured from call answer to disconnect decision. AMD typically fires within 2-3 seconds. Keyword backup may take up to 4 seconds depending on the voicemail greeting length.

**Note on BYOD/SIP calls:** For calls arriving via VICIdial's dialer, the dialer often performs its own AMD before connecting to our AI. The 4.1% voicemail rate in our data represents voicemails that passed through the dialer's detection — it is not the raw voicemail rate of the underlying phone lists.

---

## Anonymization

This dataset contains only aggregate metrics. The following information is explicitly excluded:

- Client names, company names, or campaign identifiers
- Phone numbers (caller or callee)
- Caller names or any personally identifiable information
- Transcript content
- Geographic information beyond server region
- Individual call records

Vertical names (SSDI, Final Expense, ACA) describe the type of campaign, not specific clients.

---

## Limitations

1. **Latency is server-side only.** End-to-end latency experienced by the caller includes network transit, jitter buffers, and codec delays that we do not measure. Actual perceived latency is likely 50-150ms higher than our reported numbers.

2. **AMD false positive rate is unknown.** When AMD incorrectly classifies a human as a voicemail (or vice versa), we have no direct way to measure this from our side. The detection happens at the carrier or dialer level before reaching our AI.

3. **Transfer rates depend heavily on list quality.** The same AI agent on a high-quality lead list will show higher transfer rates than on a cold list. Our benchmarks reflect the lists used by our clients during this period — your results will vary.

4. **Varying sample sizes across verticals.** SSDI (~1,450 calls) and Final Expense (~1,100 calls) provide reasonable statistical confidence for transfer rate benchmarks. ACA has the most calls (~3,400) but the fewest transfers (13), making its transfer rate less statistically robust.

5. **Two-month window.** Call center performance varies by day of week, time of day, season, and regulatory environment. A longer measurement period would provide more stable benchmarks.

6. **Single geographic region.** All infrastructure runs in US-Central. Latency benchmarks would differ for other regions.

---

## Reproducibility

These benchmarks reflect a specific configuration:

- **STT:** Deepgram Flux v2 with `eot_threshold=0.7`
- **LLM:** Llama 3.1 8B on Groq inference (streaming, temperature 0.6)
- **TTS:** Cartesia Sonic Turbo with persistent WebSocket and native mulaw output
- **Telephony:** Direct SIP registration on Asterisk 16.x (chan_sip)
- **Server:** GCP e2-standard-2 in us-central1

Changing any of these components — different STT provider, different LLM size, different TTS engine, different server location — will produce different results. These benchmarks are published as a reference point, not as guarantees.

---

## Updates

This repository will be updated as more data is collected. Each update will note the new measurement period and any changes to the AI stack or infrastructure.

Last updated: March 2026
