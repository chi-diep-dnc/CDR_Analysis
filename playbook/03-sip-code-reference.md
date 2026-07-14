# 03 — SIP Code Reference & Interpretation Methodology

## Source of truth

The definitive, verified table of SIP codes relevant to this project lives in `knowledge_base.json` → `sip_codes`. As of this writing it covers: 100, 180, 181, 183, 200, 403, 404, 486, 503, 603, 608 — each with a technical meaning, a plain-English meaning, and practical interpretation notes. **Always look up a code there first.** This document covers how to reason about the SIP protocol in general and what to do with a code that isn't in the table yet.

## The critical caveat: SIP tells you acceptance, not who answered

Per `knowledge_base.json` → `metrics_definitions.answer_rate`: SIP 200 means the network/carrier accepted the call request. It does **not** distinguish whether a human, voicemail, IVR, or answering machine picked up. That determination comes from a separate signal — answering-machine detection (AMD) or agent/dialer disposition. Never treat "SIP 200" and "reached a live person" as synonymous in analysis or reporting; always pair SIP-based answer rate with dialer-disposition-based contact/RPC rate (see [08-kpi-catalog.md](08-kpi-catalog.md)) to get the full picture.

## General SIP response classes (for codes not yet in the knowledge base)

SIP responses follow a numeric class structure. When you encounter a code not yet captured in `knowledge_base.json`, use the class to reason about it provisionally, then add it to the JSON once its specific meaning is confirmed (see "Adding a new code" below) — don't leave it as an unexplained number in an analysis.

| Class | Range | General meaning | Analytical treatment |
|---|---|---|---|
| Provisional | 1xx | Request received, processing continues | Not a final outcome — exclude from final-disposition-based KPIs; only the last response on a call matters for connect/answer rate |
| Success | 2xx | Request was successfully accepted | Generally corresponds to "answered" at the network level |
| Redirection | 3xx | Further action needed (call forwarded elsewhere) | Usually not a compliance-relevant final state on its own — check what the redirected call resolved to |
| Client Error | 4xx | The request was rejected by/for the destination (bad number, refused, busy) | Often the most compliance- and deliverability-relevant class — many spam/blocking signals surface here |
| Server Error | 5xx | A network element failed to fulfill an apparently valid request | Ambiguous — can be genuine congestion/outage OR carrier-side call blocking/spam mitigation dressed up as a server error; treat with caution, see [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md) |
| Global Failure | 6xx | The request failed everywhere it was tried (e.g., 603 Decline, 606 Not Acceptable) | Usually reflects a definitive decision by the destination or a carrier in the path |

## Adding a new SIP code to the knowledge base

When a code appears in real data that isn't yet in `knowledge_base.json` → `sip_codes`, add an entry with these fields, verified against an authoritative source (RFC 3261 for the base set, or carrier documentation for vendor-specific codes) rather than guessed from the class table alone:

```json
{
  "code": "<the 3-digit code>",
  "technical_meaning": "<the formal SIP reason phrase>",
  "plain_meaning": "<one-line plain-English translation>",
  "notes": "<practical interpretation notes, especially any ambiguity or carrier-specific nuance>"
}
```

## Platform availability caveat

Not every dialing platform surfaces SIP response codes in its CDR export — many predictive/cloud dialers expose only their own disposition vocabulary (see [04-dialer-status-taxonomy.md](04-dialer-status-taxonomy.md)) and never show the underlying SIP layer at all. Treat `sip_response_code` as an optional enrichment field: when absent, connect/not-connected classification must fall back entirely to the dialer disposition mapping, and SIP-based KPIs (like the strict SIP-200 answer rate) simply cannot be computed for that dataset — say so explicitly rather than approximating them from disposition data alone.
