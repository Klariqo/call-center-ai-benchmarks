# Call Center Voice AI Benchmarks: Real Production Data

> Open benchmarks from **49,000+** production calls processed by AI agents across SSDI, Debt Relief, ACA, Final Expense, Medicare, and Health Insurance verticals.

---

## About

These benchmarks are from a production voice AI platform processing outbound BPO calls + inbound callbacks. All data is anonymized — no client names, phone numbers, or PII. Published by [Klariqo](https://klariqo.com), a voice AI platform for call centers.

**Period:** February 2026 – July 2026 (~5.5 months, 109 active call days)
**Total calls:** 49,155
**Distinct production client deployments:** 10
**Channels:** Direct SIP (BYOD) on VICIdial Asterisk, Twilio outbound, inbound PSTN
**Verticals:** SSDI lead qualification, Debt Relief callbacks, ACA health insurance, Final Expense (outbound + inbound), Medicare benefits, general health insurance
**Update from prior version:** 1.4× the call volume of our May 2026 release (35,865 → 49,155), extends the window two more months, and adds early inbound-SMB samples. AI stack unchanged since May, so latency and cost figures carry forward.

---

## Key Findings

- **4.24% transfer rate on SSDI lead qualification** across 28,734 calls — within the upper range of experienced human agents on comparable outbound campaigns, and up from 4.04% on a smaller sample in the May release.
- **2.78% transfer rate on Debt Relief callbacks** across 7,124 calls — the vertical nearly doubled in volume since May; the rate settled slightly as the sample grew.
- **Sub-500ms response latency** on direct SIP integration (Deepgram Flux v2 STT → Gemini 2.5 Flash LLM with tool calling → Cartesia Sonic-3 TTS). Measured in the May release; the AI stack is unchanged.
- **59.8-second average duration on transferred calls** — the AI conducts nearly a minute of qualification dialogue before handoff. Median transferred-call duration: 48 seconds.
- **5.2% voicemail detection rate** with dual-method detection (carrier AMD + keyword backup), median 7.0-second detection time.
- **Direct SIP integration on VICIdial Asterisk** eliminates the Twilio relay layer, cutting 200-400ms of latency and $0.0105/min in per-call costs vs Twilio PSTN.
- **Bridge-side TEN VAD barge-in** (Layer 2) fires at ~30ms voiced speech — orders of magnitude faster than worker-side endpointing.
- **50+ concurrent calls** handled per single GCP e2-standard-2 VM (~$67/month infrastructure).
- **48,000+ calls on direct SIP (BYOD)** vs ~700 via Twilio outbound — direct SIP is the dominant production path (98.2%).

---

## 1. Call Volume and Outcome Distribution

49,155 calls processed across the production window (February–July 2026).

| Outcome | Count | Percentage | Avg Duration | Median Duration |
|---------|------:|-----------:|-------------:|----------------:|
| Disconnected | 32,846 | 66.8% | 17.7s | 16.0s |
| Abandoned / Quick Hangup | 3,713 | 7.6% | 22.1s | 16.0s |
| Other | 3,241 | 6.6% | 17.8s | 16.0s |
| Voicemail Detected | 2,573 | 5.2% | 11.4s | 7.0s |
| Transferred to Human | 1,543 | 3.1% | 59.8s | 48.0s |
| No Answer | 1,499 | 3.0% | 2.2s | 2.0s |
| Info Provided | 1,299 | 2.6% | 31.4s | 20.0s |
| Untagged | 1,045 | 2.1% | 19.3s | 16.0s |
| Not Interested | 293 | 0.6% | 26.0s | 24.0s |
| Not Qualified | 292 | 0.6% | 35.9s | 34.0s |
| Lead Captured | 289 | 0.6% | 39.4s | 33.0s |
| Completed | 143 | 0.3% | 28.6s | 27.0s |
| Complaint Logged | 135 | 0.3% | 25.5s | 23.0s |
| DNC Requested | 129 | 0.3% | 19.9s | 13.0s |
| Busy | 34 | 0.1% | 0s | 0s |
| Callback Requested | 29 | 0.1% | 33.4s | 27.0s |
| Failed | 18 | 0.04% | 0s | 0s |
| Appointment Booked | 17 | 0.03% | 76.8s | 74.0s |
| Qualified | 17 | 0.03% | 40.9s | 39.0s |

**New outcome categories this release:** Not Interested, Not Qualified, DNC Requested, Callback Requested, and Completed are broken out for the first time. In the May release these were folded into Other, so the Other bucket is smaller here even though nothing about production behavior changed.

**Context:** The 66.8% disconnected rate is typical of outbound BPO campaigns where recipients hang up early. The meaningful metric is what happens with the engaged callers — of those, the AI extracted leads, provided information, or qualified-and-transferred according to each campaign's criteria.

---

## 2. Transfer Rates by Vertical

Transfer rate is the primary success metric for outbound BPO AI — it measures how often the AI qualifies a lead well enough to warm-transfer to a human closer.

| Vertical | Total Calls | Transfers | Transfer Rate | Avg Transfer Duration |
|----------|------------:|----------:|--------------:|----------------------:|
| **SSDI Lead Qualification** | 28,734 | 1,218 | **4.24%** | ~63s |
| **Debt Relief (callbacks)** | 7,124 | 198 | **2.78%** | ~34s |
| **ACA Health Insurance** | 6,293 | 41 | 0.65% | ~72s |
| **Final Expense (callback follow-up)** | 5,791 | 66 | 1.14% | ~53s |
| **Medicare Benefits** | 667 | 12 | 1.80% | ~35s |
| **Home Services (inbound, new sample)** | 140 | 2 | 1.43% | ~70s |
| **General Health Insurance (inbound, new sample)** | 67 | 0 | 0.00% | — |
| **SSDI Lead Qualification (early-ramp sample)** | 315 | 0 | 0.00% | — |
| **Final Expense (inbound, small sample)** | 24 | 6 | 25.00% | ~259s |
| **All verticals combined** | **49,155** | **1,543** | **3.14%** | **59.8s** |

**Context on transfer rates:**

- **SSDI shows the strongest performance** at 4.24% — the qualification criteria (age, disability status, employment history) are clear enough for AI to screen efficiently, and it held up as the sample grew to 28,734 calls. Compares favorably with experienced human agents (typical 2-5% on cold outbound SSDI lists).
- **Debt Relief at 2.78%** — volume nearly doubled since May; the rate eased slightly as more calls came in, which is the normal direction when a small early sample regresses toward its stable value.
- **ACA at 0.65% and Final Expense at 1.14% are essentially unchanged** because those campaigns wound down after May — the numbers reflect little to no new volume this period rather than a new measurement.
- **Inbound channels convert dramatically higher when the caller has intent** — the 25% transfer rate on the older inbound Final Expense sample reflects callers who dialed in to engage. The two new inbound-SMB samples (Home Services, General Health Insurance) are early and low-volume; do not read a stable rate into them yet.
- **Industry comparison:** Experienced human agents on outbound BPO campaigns typically achieve 2-5% transfer rates depending on list quality, vertical, and time of day. AI's 4.24% on SSDI sits squarely in this range.

### Tracked-baseline subset

Two production deployments within this dataset are running long enough to provide stable transfer-rate baselines:

- **A long-running SSDI deployment**: 4.24% transfer rate across 28,734 calls (Feb–Jul 2026)
- **A long-running Debt Relief deployment**: 2.78% transfer rate across 7,124 calls (May–Jul 2026)

These two deployments alone represent 35,858 / 49,155 = **72.9% of total dataset volume**, so they are the strongest signal for "what stable production looks like."

---

## 3. Call Duration Analysis

Average call duration varies significantly by outcome, reflecting the depth of conversation required for each result.

| Category | Avg Duration | Median Duration | Count |
|----------|-------------:|----------------:|------:|
| All calls | 19.3s | 16.0s | 49,155 |
| Transferred | 59.8s | 48.0s | 1,543 |
| Lead Captured | 39.4s | 33.0s | 289 |
| Info Provided | 31.4s | 20.0s | 1,299 |
| Complaint Logged | 25.5s | 23.0s | 135 |
| Abandoned / Quick Hangup | 22.1s | 16.0s | 3,713 |
| Disconnected | 17.7s | 16.0s | 32,846 |
| Voicemail Detected | 11.4s | 7.0s | 2,573 |

**Key observations:**

- Transferred calls average 59.8 seconds, meaning the AI conducts substantive qualification — not just a routing exercise. Median is 48s, indicating a tight distribution around real conversation depth.
- Lead-captured calls (no transfer) average 39.4s — shorter than transfers because the AI extracts contact info + qualification answers but does not need to confirm transfer criteria.
- Voicemail detection averages 11.4s but median is 7.0s — most voicemails are caught in under 8 seconds; the 11.4s average is pulled up by edge cases where the dialer-side AMD misses and our worker-side keyword detection runs longer.

---

## 4. Channel Breakdown

| Channel | Calls | Transfers | Transfer Rate | Avg Duration | Description |
|---------|------:|----------:|--------------:|-------------:|-------------|
| **Direct SIP (BYOD)** | 48,288 | 1,527 | 3.16% | 19.2s | VICIdial dialer transfers call to AI via SIP registration |
| **Outbound (Twilio)** | 724 | 10 | 1.38% | 18.1s | AI-initiated outbound via Twilio Media Streams |
| **Inbound (PSTN)** | 142 | 6 | **4.23%** | **68.1s** | Direct inbound calls — caller dialed our number |

**Direct SIP dominates production traffic** (98.2% of calls). The Twilio outbound channel is used for clients without their own VICIdial and saw no new volume this period. Inbound calls — though only 142 in the dataset — run far longer on average (68.1s vs ~19s), reflecting engaged callers who chose to dial in. The inbound transfer rate is lower than the May release because two new inbound-SMB deployments joined the sample with few transfers so far. (One additional call ran on a legacy callback channel and is omitted from this table; it is included in the 49,155 total.)

---

## 5. Latency Benchmarks

Response latency — the time from when the caller stops speaking to when they hear the AI's first audio response — is the most critical metric for natural-sounding voice AI.

> These latency figures were measured in the May 2026 release. The AI stack (STT, LLM, TTS, bridge) is unchanged since then, so they carry forward and were not re-measured for this release.

| Metric | Value | Conditions |
|--------|------:|------------|
| **End-to-end response latency (SIP direct)** | <500ms | Direct SIP registration on VICIdial Asterisk |
| **End-to-end response latency (Twilio relay)** | 600–800ms | Twilio SIP Domain + Media Streams |
| **End-to-end response latency (HTTP API, industry estimate)** | 1,000–2,000ms | HTTP API integration mid-dialplan |
| TTS time-to-first-byte | ~40ms | Cartesia Sonic-3, persistent WebSocket, mulaw 8kHz |
| STT processing latency | 100–200ms | Deepgram Flux v2, streaming with turn detection |
| LLM inference latency (Gemini 2.5 Flash) | 80–150ms | Google AI Studio, streaming, tool calling enabled |
| LLM inference latency (Groq Llama 3.1 8B alternative) | 50–100ms | Groq cloud, streaming, lower latency / no tool calling |
| **Bridge-side barge-in (Layer 2 TEN VAD)** | **~30ms** | TEN VAD on bridge VM, voiced-speech detection |
| Worker-side barge-in (Layer 1, Deepgram VAD fallback) | 200–300ms | When TEN VAD doesn't trigger first |

### Latency breakdown (direct SIP path)

```
Caller speaks → [STT: 100-200ms] → [LLM: 80-150ms] → [TTS: 40ms] → AI responds
                                                       Total: ~220-390ms
                                                       + network overhead: <500ms
```

### Why direct SIP is faster

Most voice AI platforms integrate with call centers via one of three methods:

1. **HTTP API** (1,000–2,000ms): The dialer pauses mid-call, makes an HTTP request to an AI API, waits for audio, then plays it. High latency, audible pauses.
2. **Twilio relay** (600–800ms): Audio routes through Twilio's Media Streams WebSocket, adding ~200-400ms of relay overhead on top of AI processing.
3. **Direct SIP registration** (<500ms): The AI registers as a SIP extension on the call center's PBX (e.g., VICIdial's Asterisk). RTP audio flows directly between the PBX and the AI server — no intermediary. This is what we measured.

The 200-400ms difference between direct SIP and Twilio relay is noticeable to callers. Research on conversational turn-taking suggests gaps over 500ms feel unnatural, while gaps under 300ms feel seamless.

### Bridge-side barge-in (TEN VAD)

A second barge-in layer runs **on the SIP bridge VM** using TEN VAD (Apache 2.0, 306KB C library from Agora/TEN-framework) on every inbound µ-law RTP frame during AI playback. The bridge fires a `caller_speech_start` event at approximately 30ms of voiced speech — orders of magnitude faster than worker-side endpointing (which typically takes 200-300ms via Deepgram's VAD). The worker then cuts off AI audio playback via an in-process channel drain. Result: caller interruption feels essentially instantaneous to the human ear, not robotic.

This is a Klariqo-specific architectural advantage over voice AI platforms that rely entirely on cloud-API VAD endpointing.

---

## 6. Voicemail Detection

Voicemail detection prevents wasted AI processing time on calls that reach an answering machine.

| Metric | Value |
|--------|------:|
| Voicemail detection rate | 5.2% of all calls |
| Detection speed (median) | 7.0 seconds |
| Detection speed (average) | 11.4 seconds |
| Calls detected | 2,573 out of 49,155 |
| Action on detection | Automatic disconnect |

**Method:** Dual-layer detection combining carrier-level Answering Machine Detection (AMD) and keyword-based backup. AMD runs asynchronously at the telephony layer and typically detects within 2-3 seconds. The keyword backup catches cases AMD misses by listening for phrases like "leave a message," "not available," or "press one."

**Why the 5.2% rate is lower than industry-typical 20-40%:** Most calls are BYOD/SIP — the dialer (VICIdial) handles AMD at its level before connecting the call to our AI. The 5.2% represents voicemails that slipped past the dialer's own detection. On the outbound channels (Twilio), our voicemail detection rate is higher because we own the full detection stack.

---

## 7. Infrastructure and Capacity

| Metric | Value |
|--------|------:|
| Concurrent call capacity | 50+ per VM |
| Infrastructure (US bridge) | GCP e2-standard-2 (2 vCPU, 8GB RAM), us-central1 |
| Infrastructure cost (US bridge) | ~$67/month |
| Infrastructure (EU bridge for multi-region) | Same spec, europe-west1 |
| AI stack (primary) | Deepgram Flux v2 + Gemini 2.5 Flash (tool calling) + Cartesia Sonic-3 |
| AI stack (alternative) | Deepgram Flux v2 + Groq Llama 3.1 8B + Cartesia Sonic-3 |
| Bridge runtime | Go 1.25, single binary, systemd-managed |
| Worker runtime | TypeScript on Cloudflare Workers (edge compute) |

At 50 concurrent calls and an average duration of 19.3 seconds, a single VM can theoretically process ~9,000 calls per hour. Actual throughput depends on concurrency patterns + the ratio of long to short calls.

---

## 8. Cost Benchmarks

Per-minute costs for the AI voice pipeline (excludes telephony/carrier costs):

| Component | Cost/min | Provider | Notes |
|-----------|:--------:|----------|-------|
| Speech-to-text | $0.0077 | Deepgram Flux v2 | streaming WebSocket |
| LLM inference | ~$0.005 | Google Gemini 2.5 Flash | with tool calling; $0.001 if using Groq Llama 3.1 8B as alternative |
| Text-to-speech | $0.029 | Cartesia Sonic-3 | persistent WebSocket, native mulaw |
| **AI stack total (Gemini path)** | **~$0.042/min** | | |
| **AI stack total (Groq path)** | **~$0.038/min** | | |

Telephony costs vary by integration method:

| Method | Cost/min | Components |
|--------|:--------:|------------|
| Direct SIP (no Twilio) | $0.000 | No per-minute telephony cost — RTP direct to PBX |
| Twilio SIP inbound | $0.008 | SIP termination ($0.004) + Media Streams ($0.004) |
| Twilio PSTN inbound | $0.0125 | PSTN termination ($0.0085) + Media Streams ($0.004) |

> These are provider list prices, plan- and volume-dependent, last verified May 2026. Check each provider's current pricing page for exact figures.

---

## 9. What Changed From the May 2026 Release

| Dimension | May 2026 | This release (July 2026) |
|---|---|---|
| Total calls | 35,865 | 49,155 (**1.4×**) |
| Period | Feb–May 2026 (~3.5 months) | Feb–Jul 2026 (~5.5 months) |
| Active call days | 68 | 109 |
| SSDI transfer rate | 4.04% (775/19,203) | 4.24% (1,218/28,734) — larger sample |
| Debt Relief transfer rate | 3.06% (118/3,860) | 2.78% (198/7,124) — ~1.8× volume |
| Combined transfer rate | 2.84% (1,018/35,865) | 3.14% (1,543/49,155) |
| Direct SIP usage | 35,009 / 35,865 (97.6%) | 48,288 / 49,155 (98.2%) |
| AI stack | Deepgram Flux v2 + Gemini 2.5 Flash + Cartesia Sonic-3 | Unchanged |
| New in sample | — | Early inbound-SMB deployments (Home Services, General Health Insurance) |

Outcome categories Not Interested, Not Qualified, DNC Requested, Callback Requested, and Completed are broken out for the first time this release (previously folded into Other).

---

## Data Files

Raw benchmark data is available in CSV format:

| File | Description |
|------|-------------|
| [`data/call-duration-stats.csv`](data/call-duration-stats.csv) | Duration statistics by call outcome |
| [`data/transfer-rates.csv`](data/transfer-rates.csv) | Transfer rates by vertical |
| [`data/latency-benchmarks.csv`](data/latency-benchmarks.csv) | Response latency measurements (May 2026, stack unchanged) |
| [`data/voicemail-detection.csv`](data/voicemail-detection.csv) | Voicemail detection metrics |

---

## Methodology

See [`methodology.md`](methodology.md) for details on how each metric was measured, hardware configuration, anonymization approach, and limitations.

---

## How to Cite

If you use this data in research or publications:

```
Klariqo. "Call Center Voice AI Benchmarks: Real Production Data."
GitHub, July 2026. https://github.com/Klariqo/call-center-ai-benchmarks
```

BibTeX:

```bibtex
@misc{klariqo2026benchmarks,
  title={Call Center Voice AI Benchmarks: Real Production Data (July 2026 release)},
  author={Klariqo},
  year={2026},
  month={July},
  url={https://github.com/Klariqo/call-center-ai-benchmarks},
  note={49,155 production calls across SSDI, Debt Relief, ACA, Final Expense, Medicare, and Health Insurance verticals}
}
```

---

## About Klariqo

[Klariqo](https://klariqo.com) builds voice AI agents for BPOs, pay-per-call agencies, and call centers. The platform handles outbound calls, qualifies leads, and warm-transfers to human closers with direct VICIdial integration via SIP.

Contact: hello@klariqo.com

---

## License

[MIT License](LICENSE) — free to use, share, and build upon with attribution.
