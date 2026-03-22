# Call Center Voice AI Benchmarks: Real Production Data

> Open benchmarks from 5,900+ production outbound calls processed by AI agents.

---

## About

These benchmarks are from a production voice AI platform processing outbound BPO calls across SSDI, Final Expense, and ACA verticals. All data is anonymized — no client names, phone numbers, or PII. Published by [Klariqo](https://klariqo.com), a voice AI platform for call centers.

**Period:** February–March 2026
**Total calls:** 5,939
**Call types:** Outbound lead qualification, inbound callbacks, SIP-bridged dialer calls
**Verticals:** SSDI (Social Security Disability Insurance), Final Expense life insurance, ACA (Affordable Care Act) health insurance

---

## Key Findings

- **Sub-500ms response latency** on direct SIP integration — faster than most human agents' reaction time.
- **90.4-second average call duration** for qualified transfers, indicating substantive AI-led conversations before handoff.
- **3.5% transfer rate** on SSDI lead qualification campaigns — competitive with experienced human agents in high-volume outbound.
- **Voicemail detected in under 4 seconds** with dual-method detection (carrier AMD + keyword backup), preventing wasted AI minutes.
- **35.8% of calls abandoned within seconds** — consistent with outbound BPO industry norms where many recipients hang up immediately.
- **50+ concurrent calls** handled per single GCP e2-standard-2 VM ($67/month infrastructure).
- **Direct SIP registration** on VICIdial Asterisk eliminates the Twilio relay layer, cutting 200-400ms of latency and $0.0105/min in per-call costs.

---

## 1. Call Volume and Outcome Distribution

5,939 calls processed across three verticals over a two-month production window.

| Outcome | Count | Percentage | Avg Duration (sec) |
|---------|------:|----------:|--------------------:|
| Abandoned / Quick Hangup | 2,127 | 35.8% | 22.8 |
| Other / DNC | 1,583 | 26.6% | — |
| Info Provided | 973 | 16.4% | 29.5 |
| Voicemail Detected | 244 | 4.1% | ~0 |
| Lead Captured | 142 | 2.4% | 42.8 |
| Transferred to Human | 103 | 1.7% | 90.4 |
| Complaint | 81 | 1.4% | — |
| No Answer | 67 | 1.1% | — |
| Busy | 34 | 0.6% | — |
| Failed | 18 | 0.3% | — |
| Appointment Booked | 16 | 0.3% | — |

**Context:** The 35.8% abandonment rate is normal for outbound BPO campaigns. Many recipients hang up within the first few seconds after hearing an AI voice, especially on cold outbound. The meaningful metric is what happens with the remaining 64% — of those who stayed on the line, 16.4% received useful information, 2.4% became captured leads, and 1.7% were transferred to human agents.

### Outcome funnel

```
5,939 total calls
  ├── 2,127 abandoned (35.8%)     — recipient hung up quickly
  ├── 1,583 other/DNC (26.6%)     — do-not-call, wrong number, language barrier
  ├── 973 info provided (16.4%)   — AI answered questions, no further action needed
  ├── 244 voicemail (4.1%)        — detected and auto-disconnected
  ├── 142 leads captured (2.4%)   — contact info collected for follow-up
  ├── 103 transferred (1.7%)      — warm-transferred to human agent
  ├── 81 complaints (1.4%)        — caller expressed dissatisfaction
  ├── 67 no answer (1.1%)         — ring timeout
  ├── 34 busy (0.6%)              — busy signal
  ├── 18 failed (0.3%)            — technical failure
  └── 16 appointments (0.3%)      — appointment booked directly
```

---

## 2. Call Duration Analysis

Average call duration varies significantly by outcome, reflecting the depth of conversation required for each result.

| Category | Avg Duration | Median Duration | Count |
|----------|-------------:|----------------:|------:|
| All calls | 22.5s | 12s | 5,939 |
| Abandoned / Quick hangup | 22.8s | 8s | 2,127 |
| Qualified transfer | 90.4s | 85s | 103 |
| Lead captured | 42.8s | 38s | 142 |
| Info provided | 29.5s | 25s | 973 |
| Voicemail detected | ~0s | 0s | 244 |

**Key observations:**

- The 12-second median across all calls (vs. 22.5s average) reveals a heavily right-skewed distribution — most calls are very short, with a long tail of substantive conversations.
- Transferred calls average 90.4 seconds, meaning the AI conducts roughly 1.5 minutes of qualification conversation before deciding to hand off. This is not a simple routing exercise — the AI is asking qualification questions, confirming eligibility, and warming the lead.
- The gap between lead-captured (42.8s) and transferred (90.4s) calls suggests transfers require deeper qualification — the AI needs more conversation to confirm the lead meets transfer criteria.

### Duration by channel

| Channel | Calls | Avg Duration | Notes |
|---------|------:|-------------:|-------|
| BYOD / Direct SIP | 5,046 | 21.5s | VICIdial dialer transfers call to AI via SIP |
| Outbound / Twilio | 724 | 18.1s | AI-initiated outbound via Twilio |
| Inbound | 144 | 70.0s | Callbacks and direct inbound |

Inbound calls are dramatically longer (70s vs. ~20s) because inbound callers have intent — they called back or dialed in, so they are more likely to engage in a full conversation. Outbound and BYOD calls are short on average because a large percentage of recipients hang up immediately.

---

## 3. Transfer Rate Analysis

Transfer rate is the primary success metric for outbound BPO AI — it measures how often the AI qualifies a lead well enough to warm-transfer to a human closer.

| Vertical | Total Calls | Transfers | Transfer Rate | Avg Transfer Duration |
|----------|------------:|----------:|--------------:|----------------------:|
| SSDI Lead Qualification | ~1,450 | 51 | 3.5% | 85s |
| Final Expense Outbound | ~1,100 | 39 | 3.5% | 95s |
| ACA Health Insurance | ~3,389 | 13 | 0.4% | 88s |
| **All verticals combined** | **5,939** | **103** | **1.7%** | **90.4s** |

**Context on transfer rates:**

- The 1.7% overall rate includes all calls (answered, unanswered, voicemail, busy). When calculated against only answered and engaged calls (~3,500), the effective transfer rate is closer to 3%.
- SSDI and Final Expense both achieve 3.5% transfer rates — these are the "qualified verticals" where the AI can efficiently screen with clear eligibility criteria (age, disability status, insurance needs).
- ACA shows a lower 0.4% transfer rate. ACA campaigns in this period had higher call volumes but stricter qualification criteria (income thresholds, enrollment windows), resulting in fewer transfers per call.
- Industry comparison: Experienced human agents on outbound BPO campaigns typically achieve 2-5% transfer rates depending on list quality, vertical, and time of day. The AI's 3.5% on qualified verticals is within the competitive range for a fully automated agent.

---

## 4. Latency Benchmarks

Response latency — the time from when the caller stops speaking to when they hear the AI's first audio response — is the most critical metric for natural-sounding voice AI.

| Metric | Value | Conditions |
|--------|------:|------------|
| **End-to-end response latency (SIP direct)** | <500ms | Direct SIP registration on VICIdial Asterisk |
| **End-to-end response latency (Twilio relay)** | 600–800ms | Twilio SIP Domain + Media Streams |
| **End-to-end response latency (HTTP API)** | 1,000–2,000ms | HTTP API integration mid-dialplan (industry estimate) |
| TTS time-to-first-byte | 40ms | Cartesia Sonic Turbo, persistent WebSocket, mulaw 8kHz |
| STT processing latency | 100–200ms | Deepgram Flux v2, streaming with turn detection |
| LLM inference latency | 50–100ms | Groq cloud inference, Llama 3.1 8B, streaming |

### Latency breakdown (direct SIP path)

```
Caller speaks → [STT: 100-200ms] → [LLM: 50-100ms] → [TTS: 40ms] → AI responds
                                                        Total: ~200-350ms
                                                        + network overhead: <500ms
```

### Why direct SIP is faster

Most voice AI platforms integrate with call centers via one of three methods:

1. **HTTP API** (1,000–2,000ms): The dialer pauses mid-call, makes an HTTP request to an AI API, waits for audio, then plays it. High latency, audible pauses.
2. **Twilio relay** (600–800ms): Audio routes through Twilio's Media Streams WebSocket, adding ~200-400ms of relay overhead on top of AI processing.
3. **Direct SIP registration** (<500ms): The AI registers as a SIP extension on the call center's PBX (e.g., VICIdial's Asterisk). RTP audio flows directly between the PBX and the AI server — no intermediary. This is what we measured.

The 200-400ms difference between direct SIP and Twilio relay is perceptually significant. Research on conversational turn-taking suggests that gaps over 500ms feel unnatural, while gaps under 300ms feel seamless.

---

## 5. Voicemail Detection

Voicemail detection prevents wasted AI processing time on calls that reach an answering machine.

| Metric | Value |
|--------|------:|
| Voicemail detection rate | 4.1% of all calls |
| Detection speed | <4 seconds |
| Calls detected | 244 out of 5,939 |
| Action on detection | Automatic disconnect |

**Method:** Dual-layer detection combining carrier-level Answering Machine Detection (AMD) and keyword-based backup. AMD runs asynchronously at the telephony layer and typically detects within 2-3 seconds. The keyword backup catches cases AMD misses by listening for phrases like "leave a message" or "not available."

**Why 4.1% is low:** On outbound campaigns, voicemail rates are typically 20-40%. Our lower rate is because most calls are BYOD/SIP — the dialer (VICIdial) handles AMD at its level before connecting the call to our AI. The 4.1% represents voicemails that slipped past the dialer's own detection.

---

## 6. Infrastructure and Capacity

| Metric | Value |
|--------|------:|
| Concurrent call capacity | 50+ per VM |
| Infrastructure | GCP e2-standard-2 (2 vCPU, 8GB RAM) |
| Infrastructure cost | ~$67/month |
| Region | us-central1 |
| AI stack | Deepgram Flux v2 (STT) + Llama 3.1 8B via Groq (LLM) + Cartesia Sonic Turbo (TTS) |

At 50 concurrent calls and an average duration of 22.5 seconds, a single VM can theoretically process ~8,000 calls per hour or ~192,000 calls per day. Actual throughput depends on call patterns, concurrency peaks, and the ratio of long vs. short calls.

---

## 7. Cost Benchmarks

Per-minute costs for the AI voice pipeline (excludes telephony/carrier costs):

| Component | Cost/min | Provider |
|-----------|:--------:|----------|
| Speech-to-text | $0.0077 | Deepgram Flux v2 |
| LLM inference | $0.001 | Groq (Llama 3.1 8B) |
| Text-to-speech | $0.029 | Cartesia Sonic Turbo |
| **AI stack total** | **$0.038/min** | |

Telephony costs vary by integration method:

| Method | Cost/min | Components |
|--------|:--------:|------------|
| Direct SIP (no Twilio) | $0.000 | No per-minute telephony cost — RTP direct to PBX |
| Twilio SIP inbound | $0.008 | SIP termination ($0.004) + Media Streams ($0.004) |
| Twilio PSTN inbound | $0.0125 | PSTN termination ($0.0085) + Media Streams ($0.004) |

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
GitHub, March 2026. https://github.com/AnshumanDeb/call-center-ai-benchmarks
```

BibTeX:

```bibtex
@misc{klariqo2026benchmarks,
  title={Call Center Voice AI Benchmarks: Real Production Data},
  author={Klariqo},
  year={2026},
  month={March},
  url={https://github.com/AnshumanDeb/call-center-ai-benchmarks},
  note={5,939 production outbound calls across SSDI, Final Expense, and ACA verticals}
}
```

---

## About Klariqo

[Klariqo](https://klariqo.com) builds voice AI agents for BPOs, pay-per-call agencies, and call centers. The platform handles outbound calls, qualifies leads, and warm-transfers to human closers with direct VICIdial integration via SIP.

Contact: hello@klariqo.com

---

## License

[MIT License](LICENSE) — free to use, share, and build upon with attribution.
