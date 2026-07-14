# 04 — Dialer Status Taxonomy & Cross-Platform Mapping

## Why this exists

`knowledge_base.json` → `dialer_status` contains ~62 disposition codes captured from **one specific dialing platform**. Those exact strings (`DROP`, `PDROP`, `DNCLCC`, etc.) are not a universal standard — the next platform's export will use entirely different codes for the same underlying concepts. This document defines the **canonical categories** that any platform's codes should be bucketed into, so that connect rate, abandonment rate, and disposition-based analysis stay valid and comparable regardless of source platform.

## Canonical status categories

Every platform-specific disposition should be bucketed into exactly one of these. `connect_status` follows directly from the category.

| Category | `connect_status` | Description | Examples from the captured platform (knowledge_base.json → dialer_status) |
|---|---|---|---|
| **Line Issue** | Not Connected | The number itself is bad — disconnected, invalid, or the network rejected it as unreachable | `DC` (Disconnected Number), `FORBID` (Forbidden) |
| **No Answer** | Not Connected | The call rang out with no pickup | `NA` (No Answer AutoDial), `N` (No Answer Ringing) |
| **Busy / Congestion** | Not Connected | Line busy or network congestion prevented connection | `UB` (Busy), `B` (System Busy), `CG` (Congestion) |
| **Carrier/Network Block** | Not Connected | A carrier or spam-mitigation system blocked the call before it could reach the destination | `CIDB` (Blocked Call), `PDROP` (Pre-Routing Drop), `ROBO` (Robo Call flag) |
| **Compliance Block** | Not Connected | The call did not connect because of a consent/DNC flag applied before connection | `DNCLCC` (DNC Lead Consent Concern) |
| **Answering Machine / Voicemail** | Connected | AMD detected a machine, not a human | `AA` (Answering Machine Detected), `A` (Answering Machine) |
| **System/Queue Drop** | Connected | The call connected to the network/queue but no agent was available to take it — this is the abandonment-rate-relevant category | `DROP` (Agent Not Available In Campaign) |
| **Human — Qualified/Interested** | Connected | A live person was reached and the outcome was positive engagement | `SALE` (Xfer/Sale), `INST` (Interested) |
| **Human — Disqualified** | Connected | A live person was reached but did not qualify for the offer | The large `DQ*`/`DN*` family (`DQNB`, `DNQA`, `MINOR`, etc.) |
| **Human — Declined/Not Interested** | Connected | A live person was reached and declined | `NI` (Not Interested), `WRONG` (Wrong Number) |
| **Hang Up / Dead Air** | Connected | The call connected but ended abruptly or with no audio | `HU` (Hang Up), `DAIR` (Dead Air), `CALLHU` (Caller Hung Up) |
| **Agent/System Issue** | Connected | The call connected but was disrupted by an agent- or system-side technical problem, not the consumer's choice | `ERI` (Agent Lost Connection), `LOGOUT` (Agent Force Logout), `INCOMP` (Incomplete Call) |
| **Compliance Action** | Connected | A live interaction resulted in a compliance action being recorded | `DNC` (Do NOT Call — added to internal list) |

## Mapping procedure for a new platform

1. Pull the **complete distinct list** of disposition/status values that appear in the platform's export — not just the ones you expect. Undocumented or rarely-used codes are exactly where mapping errors hide.
2. For each raw value, read its platform documentation (or ask someone who knows the platform) for its actual meaning — do not infer meaning purely from the code's name, since abbreviations can be misleading (this project's own knowledge base already needed a correction where a code's description was ambiguous until confirmed by the user — see `knowledge_base.json` meta notes).
3. Assign each raw value to exactly one canonical category above. If none fits, that's a signal the taxonomy itself may need a new category — don't force a bad fit.
4. Derive `connect_status` from the category, not from intuition about the code name.
5. Save the finished raw-value → category mapping as a persistent artifact (e.g., a new keyed object in `knowledge_base.json`, namespaced by platform) so it doesn't have to be rebuilt for the next dataset from the same platform.

## Using this for connect/disconnect rate

Per `knowledge_base.json` → `metrics_definitions.connect_rate` and `.disconnect_rate`: these metrics are **always** derived from `connect_status`, never from eyeballing the disposition label directly, and never from SIP response codes. This holds regardless of platform — it's precisely why the canonical taxonomy above exists: so that `connect_status` means the same thing no matter which platform's raw codes fed into it.

## Handling genuinely ambiguous dispositions

Some disposition codes are inherently ambiguous about whether they represent carrier-side blocking vs. a legitimate consumer action (e.g., a 603-equivalent "Decline" disposition — see `knowledge_base.json` → `sip_codes` entry for 603, which flags exactly this ambiguity). When a disposition is ambiguous, say so in the analysis rather than picking a category arbitrarily; if the ambiguity materially affects a KPI, present both interpretations or note it as a limitation of the data.
