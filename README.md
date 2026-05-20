# Call Center Voice AI Benchmarks: Real Production Data

> Open benchmarks from **35,800+** production calls processed by AI agents across SSDI, Debt Relief, ACA, Final Expense, Medicare, and Health Insurance verticals.

---

## About

These benchmarks are from a production voice AI platform processing outbound BPO calls + inbound callbacks. All data is anonymized — no client names, phone numbers, or PII. Published by [Klariqo](https://klariqo.com), a voice AI platform for call centers.

**Period:** February 2026 – May 2026 (~3.5 months, 68 active call days)
**Total calls:** 35,865
**Distinct production client deployments:** 11
**Channels:** Direct SIP (BYOD) on VICIdial Asterisk, Twilio outbound, inbound PSTN
**Verticals:** SSDI lead qualification, Debt Relief callbacks, ACA health insurance, Final Expense (outbound + inbound), Medicare benefits, general health insurance
**Update from prior version:** 6.0× the call volume of our March 2026 release (5,939 → 35,865), adds Debt Relief + Medicare verticals, refreshes AI stack to current production configuration.

---

## Key Findings

- **4.04% transfer rate on SSDI lead qualification** across 19,200 calls — within the upper range of experienced human agents on comparable outbound campaigns.
- **3.06% transfer rate on Debt Relief callbacks** across 3,860 calls — new vertical added since the March release.
- **Sub-500ms response latency** on direct SIP integration (Deepgram Flux v2 STT → Gemini 2.5 Flash LLM with tool calling → Cartesia Sonic-3 TTS).
- **69-second average duration on transferred calls** — the AI conducts over a minute of qualification dialogue before handoff. Median transferred-call duration: 54 seconds.
- **6.0% voicemail detection rate** with dual-method detection (carrier AMD + keyword backup), median 7.7-second detection time.
- **Direct SIP integration on VICIdial Asterisk** eliminates the Twilio relay layer, cutting 200-400ms of latency and $0.0105/min in per-call costs vs Twilio PSTN.
- **Bridge-side TEN VAD barge-in** (Layer 2, deployed April 2026) fires at ~30ms voiced speech — orders of magnitude faster than worker-side endpointing.
- **50+ concurrent calls** handled per single GCP e2-standard-2 VM (~$67/month infrastructure).
- **35,000+ calls on direct SIP (BYOD)** vs ~700 via Twilio outbound — direct SIP is the dominant production path.

---

## 1. Call Volume and Outcome Distribution

35,865 calls processed across six verticals over a ~3.5-month production window.

| Outcome | Count | Percentage | Avg Duration | Median Duration |
|---------|------:|-----------:|-------------:|----------------:|
| Disconnected | 22,932 | 63.9% | 18.5s | 16.3s |
| Abandoned / Quick Hangup | 3,709 | 10.3% | 22.1s | 16.3s |
| Other | 3,202 | 8.9% | 17.6s | 16.1s |
| Voicemail Detected | 2,164 | 6.0% | 11.5s | 7.7s |
| Info Provided | 1,301 | 3.6% | 31.5s | 20.1s |
| Transferred to Human | 1,018 | 2.8% | 69.0s | 54.2s |
| Untagged | 996 | 2.8% | 18.6s | 16.1s |
| Lead Captured | 273 | 0.8% | 40.3s | 34.3s |
| Complaint Logged | 134 | 0.4% | 25.4s | 22.5s |
| No Answer | 67 | 0.2% | 0s | 0s |
| Busy | 34 | 0.1% | 0s | 0s |
| Failed | 18 | 0.05% | 0s | 0s |
| Appointment Booked | 2 | trace | 21.2s | 21.2s |

**Context:** The 63.9% disconnected rate is typical of outbound BPO campaigns where recipients hang up early. The meaningful metric is what happens with the engaged ~30% of callers — of those, the AI extracted leads, provided information, or qualified-and-transferred according to each campaign's criteria.

---

## 2. Transfer Rates by Vertical

Transfer rate is the primary success metric for outbound BPO AI — it measures how often the AI qualifies a lead well enough to warm-transfer to a human closer.

| Vertical | Total Calls | Transfers | Transfer Rate | Avg Transfer Duration |
|----------|------------:|----------:|--------------:|----------------------:|
| **SSDI Lead Qualification** | 19,203 | 775 | **4.04%** | ~70s |
| **Debt Relief (callbacks)** | 3,860 | 118 | **3.06%** | ~55s |
| **ACA Health Insurance** | 6,287 | 41 | 0.65% | ~75s |
| **Final Expense (callback follow-up)** | 5,791 | 66 | 1.14% | ~60s |
| **Medicare Benefits** | 608 | 7 | 1.15% | ~50s |
| **Final Expense (inbound, small sample)** | 24 | 6 | 25.00% | ~167s |
| **General Health Insurance (small sample)** | 71 | 1 | 1.41% | ~52s |
| **All verticals combined** | **35,865** | **1,018** | **2.84%** | **69.0s** |

**Context on transfer rates:**

- **SSDI shows the strongest performance** at 4.04% — the qualification criteria (age, disability status, employment history) are clear enough for AI to screen efficiently. Compares favorably with experienced human agents (typical 2-5% on cold outbound SSDI lists).
- **Debt Relief at 3.06%** — added as a vertical post-March 2026 release. Strong transfer rate for a callback-driven campaign where leads previously filled out a web form.
- **ACA at 0.65%** — lower transfer rate reflects stricter qualification windows (enrollment periods, income thresholds) AND ACA campaign list dynamics, not AI capability. Consistent with our March release's 0.4% ACA finding.
- **Inbound channels convert dramatically higher** — the 25% transfer rate on the inbound Final Expense sample reflects intent: callers who dialed in chose to engage. Sample size is small but the pattern matches our wider observations.
- **Industry comparison:** Experienced human agents on outbound BPO campaigns typically achieve 2-5% transfer rates depending on list quality, vertical, and time of day. AI's 4.04% on SSDI and 3.06% on Debt Relief sit squarely in this range.

### Tracked-baseline subset

Two production deployments within this dataset are running long enough to provide stable transfer-rate baselines:

- **A long-running SSDI deployment**: 4.10% transfer rate across 18,888 calls (Feb–May 2026)
- **A long-running Debt Relief deployment**: 3.06% transfer rate across 3,860 calls (May 2026)

These two deployments alone represent 22,748 / 35,865 = **63.4% of total dataset volume**, so they are the strongest signal for "what stable production looks like."

---

## 3. Call Duration Analysis

Average call duration varies significantly by outcome, reflecting the depth of conversation required for each result.

| Category | Avg Duration | Median Duration | Count |
|----------|-------------:|----------------:|------:|
| All calls | 19.7s | 16.3s | 35,865 |
| Transferred | 69.0s | 54.2s | 1,018 |
| Lead Captured | 40.3s | 34.3s | 273 |
| Info Provided | 31.5s | 20.1s | 1,301 |
| Complaint Logged | 25.4s | 22.5s | 134 |
| Abandoned / Quick Hangup | 22.1s | 16.3s | 3,709 |
| Disconnected | 18.5s | 16.3s | 22,932 |
| Voicemail Detected | 11.5s | 7.7s | 2,164 |

**Key observations:**

- Transferred calls average 69 seconds, meaning the AI conducts substantive qualification — not just a routing exercise. Median is 54s, indicating a tight distribution around real conversation depth.
- Lead-captured calls (no transfer) average 40.3s — shorter than transfers because the AI extracts contact info + qualification answers but does not need to confirm transfer criteria.
- Voicemail detection averages 11.5s but median is 7.7s — most voicemails are caught in under 8 seconds; the 11.5s average is pulled up by edge cases where the dialer-side AMD misses and our worker-side keyword detection runs longer.

---

## 4. Channel Breakdown

| Channel | Calls | Transfers | Transfer Rate | Avg Duration | Description |
|---------|------:|----------:|--------------:|-------------:|-------------|
| **Direct SIP (BYOD)** | 35,009 | 989 | 2.82% | 20.2s | VICIdial dialer transfers call to AI via SIP registration |
| **Outbound (Twilio)** | 724 | 10 | 1.38% | 18.1s | AI-initiated outbound via Twilio Media Streams |
| **Inbound (PSTN)** | 110 | 9 | **8.18%** | **75.1s** | Direct inbound calls — caller dialed our number |

**Direct SIP dominates production traffic** (97.6% of calls). The Twilio outbound channel is used for clients without their own VICIdial. Inbound calls — though only 110 in the dataset — show **2.9× the transfer rate** of outbound. This reflects intent: inbound callers chose to engage, while outbound callers were dialed.

---

## 5. Latency Benchmarks

Response latency — the time from when the caller stops speaking to when they hear the AI's first audio response — is the most critical metric for natural-sounding voice AI.

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
| Voicemail detection rate | 6.0% of all calls |
| Detection speed (median) | 7.7 seconds |
| Detection speed (average) | 11.5 seconds |
| Calls detected | 2,164 out of 35,865 |
| Action on detection | Automatic disconnect |

**Method:** Dual-layer detection combining carrier-level Answering Machine Detection (AMD) and keyword-based backup. AMD runs asynchronously at the telephony layer and typically detects within 2-3 seconds. The keyword backup catches cases AMD misses by listening for phrases like "leave a message," "not available," or "press one."

**Why the 6.0% rate is lower than industry-typical 20-40%:** Most calls are BYOD/SIP — the dialer (VICIdial) handles AMD at its level before connecting the call to our AI. The 6.0% represents voicemails that slipped past the dialer's own detection. On the outbound channels (Twilio), our voicemail detection rate is higher because we own the full detection stack.

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

At 50 concurrent calls and an average duration of 19.7 seconds, a single VM can theoretically process ~9,000 calls per hour. Actual throughput depends on concurrency patterns + the ratio of long to short calls.

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

---

## 9. What Changed From the March 2026 Release

| Dimension | March 2026 | This release (May 2026) |
|---|---|---|
| Total calls | 5,939 | 35,865 (**6.0×**) |
| Period | Feb–Mar 2026 (~2 months) | Feb–May 2026 (~3.5 months) |
| Verticals covered | 3 (SSDI, FE, ACA) | 6 (added Debt Relief, Medicare, general Health Insurance) |
| LLM | Llama 3.1 8B on Groq (only) | Gemini 2.5 Flash primary (tool calling); Groq Llama 3.1 8B as alternative |
| TTS | Cartesia Sonic Turbo | Cartesia Sonic-3 |
| Barge-in | Worker-side Deepgram VAD (~200-300ms) | + Bridge-side TEN VAD (Layer 2, ~30ms) |
| Direct SIP usage | 5,046 / 5,939 (85.0%) | 35,009 / 35,865 (97.6%) — proved out as dominant path |
| SSDI transfer rate | 3.5% (51/1,450) | 4.04% (775/19,203) — 13.2× larger sample |
| Debt Relief vertical | Not measured | 3.06% (118/3,860) — new vertical |

---

## Data Files

Raw benchmark data is available in CSV format:

| File | Description |
|------|-------------|
| [`data/call-duration-stats.csv`](data/call-duration-stats.csv) | Duration statistics by call outcome |
| [`data/transfer-rates.csv`](data/transfer-rates.csv) | Transfer rates by vertical |
| [`data/latency-benchmarks.csv`](data/latency-benchmarks.csv) | Response latency measurements |
| [`data/voicemail-detection.csv`](data/voicemail-detection.csv) | Voicemail detection metrics |

---

## Methodology

See [`methodology.md`](methodology.md) for details on how each metric was measured, hardware configuration, anonymization approach, and limitations.

---

## How to Cite

If you use this data in research or publications:

```
Klariqo. "Call Center Voice AI Benchmarks: Real Production Data."
GitHub, May 2026. https://github.com/Klariqo/call-center-ai-benchmarks
```

BibTeX:

```bibtex
@misc{klariqo2026benchmarks,
  title={Call Center Voice AI Benchmarks: Real Production Data (May 2026 release)},
  author={Klariqo},
  year={2026},
  month={May},
  url={https://github.com/Klariqo/call-center-ai-benchmarks},
  note={35,865 production calls across SSDI, Debt Relief, ACA, Final Expense, Medicare, and Health Insurance verticals}
}
```

---

## About Klariqo

[Klariqo](https://klariqo.com) builds voice AI agents for BPOs, pay-per-call agencies, and call centers. The platform handles outbound calls, qualifies leads, and warm-transfers to human closers with direct VICIdial integration via SIP.

Contact: hello@klariqo.com

---

## License

[MIT License](LICENSE) — free to use, share, and build upon with attribution.
