# 09 — Analytical Methodologies

General approaches for turning normalized CDR data + the KPI catalog into actual findings.

## Funnel analysis

Model every outbound campaign as a funnel of canonical stages, and always report both the count and the stage-to-stage conversion rate:

```
Dialed → Connected → Human Contact → Right Party Contact → Transferred → Converted/Sold
```

- Each arrow is a KPI from [08-kpi-catalog.md](08-kpi-catalog.md) (Connect Rate, Contact Rate, RPC Rate, Transfer Rate, Conversion Rate).
- **Abnormal drop-off localization:** compare each stage-to-stage ratio against its own historical baseline (same campaign, prior period) rather than an absolute benchmark — what's normal varies enormously by vertical/list type. A drop concentrated at one specific stage (e.g., Connected → Human Contact falling while Dialed → Connected stays flat) points to a different root cause than a drop at Dialed → Connected (the former suggests AMD/voicemail-heavy lists or answer-but-hangup behavior; the latter suggests carrier-side blocking or list quality).

## Time-series / trend analysis

- Track KPIs daily (at minimum) and roll up to the windows the metric definition requires (e.g., abandonment rate's mandatory 30-day window — never substitute a shorter window and call it the same metric).
- Use control-chart thinking: establish a baseline mean and typical variance for a KPI, then flag points outside normal variance rather than reacting to every day-to-day wiggle.
- Overlay known operational events (new number pool launch, campaign list change, carrier change) on the timeline before attributing a trend shift to an external cause like carrier behavior — rule out self-inflicted causes first.

## Segmentation analysis

Break down any headline KPI along these dimensions before drawing conclusions, since an aggregate number can hide a problem concentrated in one slice:

- **Campaign / list**
- **ANI / caller ID / number pool**
- **Destination state / timezone** (also required for compliance analysis)
- **Carrier** (originating and, if known, terminating)
- **Dialer platform**, if multiple are in use
- **Time-of-day / day-of-week**
- **STIR/SHAKEN attestation level**, if available

A KPI that looks fine in aggregate but is being dragged up by a large well-performing segment while a smaller segment is badly degraded is a common and important finding — always check at least the ANI and state/timezone breakdowns before declaring an aggregate KPI "healthy."

## Root-cause decomposition method

When a KPI moves meaningfully, work through dimensions systematically rather than jumping to a conclusion:

1. **Isolate the scope.** Does the shift appear across all campaigns/numbers, or is it concentrated? Segment (above) until you find the smallest group that explains most of the shift.
2. **Check the timeline against known changes.** New numbers added? Warm-up followed (see [06-deliverability-best-practices.md](06-deliverability-best-practices.md))? Campaign/list changed? Carrier/route changed?
3. **Check the disposition/SIP-code mix shift**, not just the headline rate. A connect-rate drop driven by a spike in SIP 608 tells a different story (spam labeling — see [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md)) than one driven by a spike in plain no-answer (list/timing issue) or busy/congestion (network issue).
4. **Cross-check volume consistency** for the affected number(s) — a self-inflicted volume spike is a very common cause of an apparent "carrier is blocking us" pattern.
5. **State the conclusion with confidence level.** If the CDR data plus available context clearly points to one cause, say so. If it's genuinely ambiguous between two explanations (e.g., 503 spike = congestion vs. soft-block), say that explicitly rather than picking one.

## Anomaly detection heuristics

- Sudden concentration of a single SIP or dialer-disposition code where the historical mix was diverse.
- Daily volume for an ANI outside its own ±70–100% 14-day band.
- Connect-rate drop that begins within days of a number pool's first-use date (points to inadequate warm-up).
- Abandonment rate crossing 3% for any campaign in the trailing 30-day window (a hard compliance threshold, not just a statistical anomaly).
- Divergence between SIP-based answer rate and disposition-based contact rate widening over time (may indicate a labeling issue suppressing genuine engagement even where calls are technically "answered").

## Cohort / A-B comparison

When comparing number pools, campaigns, or caller-ID strategies, hold the comparison to like-for-like conditions: same date range, same destination-geography mix if possible, and same campaign/list quality — a raw KPI comparison between two number pools dialing different lists at different times is not a valid A/B comparison, and any finding based on it should note that confound explicitly.
