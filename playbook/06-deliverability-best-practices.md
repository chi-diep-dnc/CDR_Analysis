# 06 — Deliverability Best Practices

This restates `knowledge_base.json` → `call_deliverability_guide` (sourced from DNC.com's Call Deliverability Guide) as an actionable checklist for analysts auditing a calling program's deliverability health. For exact source wording/thresholds, always refer back to the JSON — this document is the "how to check it" layer.

## Deliverability health checklist

Run this checklist against any number pool or campaign under investigation:

| Check | Threshold / Rule (from `knowledge_base.json`) | How to verify in CDR data |
|---|---|---|
| Call abandonment rate | ≤ 3% per campaign, measured over a trailing 30-day period (`call_abandonment_rate`) | `count(dialer_disposition_canonical == "System/Queue Drop") / count(connect_status == "Connected")`, rolled up over 30 days per campaign |
| Ring time before disconnect | ≥ 15 seconds or 4 rings (`call_ring_time`) | Check `duration_total_sec` for calls disposed as no-answer — flag any systematically shorter than this |
| Agent pickup latency | Recorded/callback message played if no agent within 2 seconds of greeting completion (`connectivity_requirements`) | Requires agent-connect-latency field if available; otherwise infer from abandonment-rate trend |
| Per-number daily volume consistency | Within roughly ±70–100% of that number's trailing 14-day average (`call_volume_consistency`) | Compute daily call count per ANI, compare to its own 14-day rolling average; flag outliers |
| No hourly "catch-up" bursts | Calls paced evenly across business hours, not concentrated at day's end (`call_volume_consistency`) | Plot call volume by hour-of-day per number; flag sawtooth/end-of-day spike patterns |
| No caller ID sharing across sellers/brands | One caller ID per seller/brand, not reused across unrelated campaigns (`caller_id`) | Cross-reference ANI usage against campaign/brand metadata |
| New number warm-up | 5–10 internal test calls, or 2–3 days of activity per major carrier, before full-volume use (`warm_up_new_numbers`) | Check first-use date of each ANI against its early volume ramp |
| Caller ID substitution rules | If substituting, must show seller's name + a customer-service number staffed during business hours (`caller_id`) | Requires caller-ID-display field; not always available in CDR alone — flag as a policy check outside CDR scope if unavailable |

## Interpreting a failed check

A failed check on this list is a **leading indicator**, not proof of a labeling event — pair it with the diagnostic signal table in [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md) and the connect-rate/SIP-code trend for the same number pool before concluding causation. Multiple failed checks co-occurring with a connect-rate or SIP-608 trend is a much stronger signal than any single check alone.

## Relationship to compliance

Several of these checks (abandonment rate, ring time) are simultaneously **compliance requirements**, not just deliverability optimizations — see [07-compliance-and-legal-considerations.md](07-compliance-and-legal-considerations.md). Don't treat a failed abandonment-rate check as purely a "spam risk" issue; it is also a regulatory exposure in its own right.
