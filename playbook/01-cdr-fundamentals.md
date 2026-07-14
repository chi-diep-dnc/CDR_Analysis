# 01 — CDR Fundamentals

## What a Call Detail Record is

A Call Detail Record (CDR) is a single log entry describing one call attempt: who/what initiated it, what number it targeted, when it happened, how the network responded, how long it lasted, and how it was ultimately disposed of (by a carrier, a dialer system, or a human agent). Every downstream metric in this playbook — connect rate, abandonment rate, spam-labeling risk — is a rollup over collections of CDRs.

## The call lifecycle

A single outbound call attempt typically passes through these stages, though not every platform's CDR exposes every stage:

1. **Origination** — the dialer/switch initiates the call attempt, selecting an outbound caller ID (ANI) and a destination number (DNIS).
2. **Network signaling** — the call is routed through one or more carriers. If SIP is exposed at this layer, provisional responses (100 Trying, 180 Ringing, 183 Session Progress) may appear before a final response.
3. **Termination outcome** — the call reaches a final state: answered (SIP 200), rejected/blocked (403, 608), unreachable (404), busy (486), or a network/server condition (503).
4. **Connection handling** — if answered, the call is handled by: an IVR, answering-machine detection (AMD), a live agent, or a prerecorded message. This is where "who/what actually answered" is determined — a detail the SIP layer alone cannot tell you (see [03-sip-code-reference.md](03-sip-code-reference.md)).
5. **Disposition** — the dialer platform or agent assigns a disposition/status code describing the outcome from a business perspective (e.g., sale, not interested, wrong number, DNC request). This is the dialer status layer (see [04-dialer-status-taxonomy.md](04-dialer-status-taxonomy.md)).
6. **Termination** — the call ends, and duration/talk-time fields are finalized.

## Why platform diversity is the central problem

CDRs from different outbound dialing platforms (predictive dialers, cloud contact-center platforms, SIP trunking providers) describe the same underlying events using different field names, different status vocabularies, and different levels of granularity. A record that one platform calls `DISPOSITION` might be split across `hangup_cause` and `agent_disposition` on another. A "connected" call on one platform's status list might use the code `HU` (Hang Up) while another uses `CONNECTED_HANGUP`.

This means CDR analysis cannot be done by memorizing one platform's field names — it must be done by:

1. Mapping every platform's raw export into the **canonical schema** ([02-canonical-data-schema.md](02-canonical-data-schema.md)), and
2. Mapping every platform's raw status/disposition codes into the **canonical status taxonomy** ([04-dialer-status-taxonomy.md](04-dialer-status-taxonomy.md)),

...before computing any KPI. Any KPI computed directly on raw, unmapped platform fields risks being wrong or incomparable across data sources.

## Two distinct kinds of CDR

Be careful not to conflate these — they often live in separate exports and need to be joined by call ID or timestamp+ANI+DNIS:

- **Dialer/campaign CDRs** — produced by the outbound dialing platform. Rich in business context (campaign, agent, disposition, list membership) but may or may not expose the raw SIP response.
- **Carrier/SIP CDRs** — produced by the underlying telephony/SIP trunk provider. Rich in network-level detail (SIP response codes, call duration at the trunk level, sometimes STIR/SHAKEN attestation) but has no knowledge of campaign/agent/business disposition.

A complete analysis usually requires both, joined together. When only one is available, note explicitly in the analysis which questions cannot be answered (e.g., dialer-only data cannot answer "was this a carrier-side block or a network failure").
