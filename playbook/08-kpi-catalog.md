# 08 — KPI Catalog

All formulas below use canonical field names from [02-canonical-data-schema.md](02-canonical-data-schema.md). Compute every KPI only after normalization (raw platform fields mapped to canonical schema, dispositions mapped to the taxonomy in [04-dialer-status-taxonomy.md](04-dialer-status-taxonomy.md)) — a KPI computed on unmapped raw data is not trustworthy and should not be reported.

## Volume & connection KPIs

### Dial Volume (Attempts)
- **Formula:** `COUNT(*)` over the period/segment in question.
- **Required fields:** none beyond a row per attempt.
- **Interpretation:** the denominator for nearly every other rate below. Always report alongside rates, never a rate alone.
- **Pitfalls:** watch for duplicate records (retries logged twice) and for redial attempts on the same lead being conflated with unique leads dialed — decide and state which you're measuring.

### Connect Rate
- **Formula:** `COUNT(connect_status == "Connected") / COUNT(*)`
- **Required fields:** `connect_status` (derived from `dialer_disposition_canonical`).
- **Interpretation:** per `knowledge_base.json` → `metrics_definitions.connect_rate` — always derived from `connect_status`, never from eyeballing the raw disposition label, since dispositions can change over time but the connect/not-connected classification is the stable source of truth.
- **Pitfalls:** mixing platforms with different disposition vocabularies without going through the taxonomy mapping first will silently corrupt this number.

### Disconnect Rate
- **Formula:** `1 - Connect Rate`, or `COUNT(connect_status == "Not Connected") / COUNT(*)`
- **Interpretation:** per `metrics_definitions.disconnect_rate`. Same sourcing rule as Connect Rate.

### Answer Rate (SIP-based)
- **Formula:** `COUNT(sip_response_code == 200) / COUNT(*)`
- **Required fields:** `sip_response_code` — **not available on all platforms** (see [03-sip-code-reference.md](03-sip-code-reference.md)); when absent, this KPI cannot be computed and should be reported as N/A, not approximated from disposition data.
- **Interpretation:** per `metrics_definitions.answer_rate` — measures network-level acceptance only, not who/what answered. Always report alongside a contact/RPC rate (below) for the full picture.

## Contact quality KPIs

### Answering Machine Detection (AMD) Rate
- **Formula:** `COUNT(dialer_disposition_canonical == "Answering Machine / Voicemail") / COUNT(connect_status == "Connected")`
- **Interpretation:** share of connected calls that hit a machine rather than a live line. A rising AMD rate with a stable connect rate suggests list/number quality shifting toward voicemail-heavy destinations, not a deliverability problem per se.

### Contact Rate (Live Human Reached)
- **Formula:** `COUNT(dialer_disposition_canonical IN {Human — Qualified/Interested, Human — Disqualified, Human — Declined/Not Interested}) / COUNT(*)`
- **Interpretation:** the share of all dial attempts that reached an actual person, as opposed to just the network accepting the call.

### Right Party Contact (RPC) Rate
- **Formula:** `COUNT(confirmed correct-person human contact) / COUNT(connect_status == "Connected")`
- **Required fields:** requires a disposition category (or agent note) that confirms the *correct* person was reached, not just *a* person — plain "Human — Declined/Not Interested" often includes wrong-number cases (e.g., the captured platform's `WRONG` code), which should be excluded from RPC even though they're a live human contact.
- **Pitfalls:** conflating "any human contact" with "right party contact" overstates list quality.

### Transfer / Sale Rate
- **Formula:** `COUNT(transfer_flag == true) / COUNT(connect_status == "Connected")` (or `/ RPC count`, state which denominator is used)
- **Interpretation:** the ultimate funnel outcome for most outbound sales/lead-gen campaigns.

### Conversion Rate
- **Formula:** `COUNT(disposition == final positive outcome, e.g. SALE) / COUNT(*)` (total-attempt basis) — distinct from Transfer Rate if "transferred" and "sold" are different pipeline stages.

## Deliverability & compliance KPIs

### Abandonment Rate
- **Formula:** `COUNT(dialer_disposition_canonical == "System/Queue Drop") / COUNT(connect_status == "Connected")`, computed **per campaign, over a trailing 30-day window**.
- **Compliance threshold:** ≤ 3% (`knowledge_base.json` → `call_deliverability_guide.call_abandonment_rate`).
- **Pitfalls:** computing this over too short a window (a single day) can produce noisy, non-compliance-representative figures — the 30-day window is a defined part of the metric, not an arbitrary choice.

### Carrier Block/Reject Rate (by SIP class)
- **Formula:** `COUNT(sip_response_code IN {403, 503, 608}) / COUNT(*)`, broken out per-code as well as combined.
- **Interpretation:** see the diagnostic table in [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md) for how to read shifts in this metric.

### Compliance Calling-Hours Violation Rate
- **Formula:** `COUNT(timestamp_start outside the applicable legal window for destination_state/timezone) / COUNT(*)`
- **Required fields:** resolved destination timezone (see [07-compliance-and-legal-considerations.md](07-compliance-and-legal-considerations.md) workflow step 1–2).
- **Pitfalls:** this KPI is only as reliable as the underlying timezone resolution — flag low-confidence resolutions (see the `zip3_reliable` caveat) rather than presenting a violation count with false precision.

### Volume Consistency Deviation
- **Formula:** for each ANI, `(today's volume - trailing 14-day average) / trailing 14-day average`
- **Threshold:** flagged outside roughly ±70–100% (`call_deliverability_guide.call_volume_consistency`).
- **Interpretation:** a leading indicator for spam-labeling risk, not itself a compliance metric.

## Operational KPIs

### Average Talk Time / Average Handle Time
- **Formula:** `AVG(duration_talk_sec)` over connected calls (talk time), or `AVG(duration_total_sec)` (handle time, dial-to-hangup).

### Average Ring Time / Time-to-Answer
- **Formula:** `AVG(timestamp_answer - timestamp_start)` for connected calls.
- **Compliance cross-check:** unanswered calls disconnected before 15 seconds/4 rings violate `call_ring_time` (see [06-deliverability-best-practices.md](06-deliverability-best-practices.md)).

### Number Health Score (composite, illustrative)
- **Formula (suggested starting point, not a fixed standard):** a weighted combination of connect-rate trend, abandonment-rate compliance, and volume-consistency deviation for a given ANI, normalized to a 0–100 scale.
- **Note:** this is a composite metric the analyst should define explicitly for a given engagement (document the exact weights used) rather than treating as a fixed formula — unlike the metrics above, it's not sourced from `knowledge_base.json`.

## Reporting convention

Every KPI reported should state: the exact formula/fields used, the time window, the segment (campaign/ANI/state/platform), and — for anything touching compliance or timezone resolution — the confidence level of the underlying geography/timezone data. Never report a bare percentage without these.
