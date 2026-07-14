# 10 — Investigation Workflows

Structured runbooks for common investigation triggers. Each follows: **Trigger → Hypotheses → Data to Pull → Diagnostic Checks → Likely Causes & Actions → Escalation**.

---

## Workflow 1: Connect Rate Dropped Significantly

- **Trigger:** Connect rate for a campaign/pool falls meaningfully below its recent baseline.
- **Hypotheses:** carrier-side blocking/spam labeling; list quality degradation; new unwarmed numbers; network/congestion issue; genuine change in destination population (e.g., new list of worse numbers).
- **Data to pull:** connect rate segmented by ANI, state, carrier, time-of-day ([09-analytical-methodologies.md](09-analytical-methodologies.md)); SIP/disposition code mix shift; ANI first-use dates; daily volume per ANI.
- **Diagnostic checks:**
  1. Is the drop concentrated in specific ANIs? → check warm-up compliance and volume consistency for those ANIs ([06-deliverability-best-practices.md](06-deliverability-best-practices.md)).
  2. Is there a spike in SIP 608/403/503? → see the carrier diagnostic table ([05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md)).
  3. Is the drop concentrated by destination state/carrier rather than by originating ANI? → points away from a labeling issue on your numbers and toward a destination-side or route-specific issue.
  4. Did the list/campaign source change recently? → rule out simple list-quality change before concluding a deliverability issue.
- **Likely causes & actions:** if isolated to new/unwarmed ANIs — pause and properly warm them. If correlated with volume spikes — smooth volume back within the consistency band. If broad SIP 608 spike with no self-inflicted cause found — treat as probable spam-labeling event; consider caller-ID reputation remediation.
- **Escalation:** if no CDR-visible cause is found and the drop is severe/sustained, escalate for carrier-side investigation (this is generally beyond what CDR data alone can resolve).

---

## Workflow 2: Spike in Spam-Likely Labeling / SIP 608

- **Trigger:** noticeable rise in SIP 608 (Rejected/Spam-like) rate.
- **Hypotheses:** volume spike on one or more ANIs; inadequate number warm-up; caller-ID reuse/sharing violation; degraded STIR/SHAKEN attestation mix; genuine external mislabeling.
- **Data to pull:** per-ANI daily volume vs. 14-day average; ANI first-use dates; attestation-level mix over time, if available; caller-ID-to-campaign mapping.
- **Diagnostic checks:** cross-reference every check in the deliverability checklist ([06-deliverability-best-practices.md](06-deliverability-best-practices.md)) for the affected ANIs specifically.
- **Likely causes & actions:** most commonly traced to a volume-consistency violation or missed warm-up on newly added numbers. Remediate the specific violated practice; note that carrier reputation recovery is typically not immediate even after the underlying cause is fixed.
- **Escalation:** if all checklist items pass and labeling persists, escalate to carrier/analytics-provider remediation channels (outside CDR-analysis scope).

---

## Workflow 3: Abandonment Rate Exceeds 3% Compliance Threshold

- **Trigger:** the 30-day trailing abandonment rate for a campaign crosses 3% (`knowledge_base.json` → `call_deliverability_guide.call_abandonment_rate`).
- **Hypotheses:** under-staffed campaign relative to dial pace (predictive dialer over-pacing); agent availability issue; a spike in connect volume outpacing agent capacity.
- **Data to pull:** `System/Queue Drop`-category disposition counts per day, connected-call volume per day, agent headcount/schedule if available.
- **Diagnostic checks:** is the abandonment concentrated at specific times of day (pacing/staffing mismatch) or spread evenly (systemic over-dialing)?
- **Likely causes & actions:** this is both a deliverability and a **compliance** issue — flag it as such, not merely an operational inefficiency. Recommend dialer pacing adjustment or staffing increase.
- **Escalation:** sustained non-compliance should be escalated immediately given regulatory exposure — this is not a "monitor and see" situation.

---

## Workflow 4: Suspected Calling-Time (Legal Hours) Violation

- **Trigger:** a call or batch of calls is suspected to have occurred outside legal calling hours for the destination.
- **Hypotheses:** timezone misresolution (especially in a `zip3_reliable: false` state); genuine scheduling/dialer misconfiguration; DST handling error.
- **Data to pull:** `timestamp_start`, `destination_zip`/`destination_state`, resolved timezone and DST status.
- **Diagnostic checks:** follow the full compliance-check workflow in [07-compliance-and-legal-considerations.md](07-compliance-and-legal-considerations.md) step 1–2. Explicitly check whether the destination state is one of the 15 flagged `zip3_reliable: false` states before trusting a ZIP3-based timezone resolution — if so, treat the finding as lower-confidence until verified at county/ZIP5 level.
- **Likely causes & actions:** if traced to a timezone-resolution error, fix the resolution method (prefer NPA-NXX) rather than the dialer schedule. If traced to genuine dialer misconfiguration, that's a direct compliance fix.
- **Escalation:** any confirmed violation should be escalated per the organization's compliance-incident process; this playbook does not define that process.

---

## Workflow 5: Carrier Appears to Be Blocking Calls (403/503 spike)

- **Trigger:** rise in SIP 403 or 503 responses.
- **Hypotheses:** genuine network congestion/maintenance; carrier-side soft-blocking dressed as a server error; destination-side rejection (legitimate declines at scale, e.g. a bad list).
- **Diagnostic checks:** is the spike carrier-specific/route-specific (points to a specific carrier relationship or congestion) or spread across all carriers for one originating ANI (points to that ANI's reputation)? Is it destination-region-specific (points to a regional network issue) or ANI-specific?
- **Likely causes & actions:** route the finding to the right owner based on the concentration pattern — carrier relationship issue vs. ANI reputation issue vs. list quality issue require entirely different fixes.
- **Escalation:** carrier-specific route issues typically require direct carrier engagement, outside CDR-analysis scope — hand off with the segmented evidence.

---

## Workflow 6: Answer Rate High but Conversion Low

- **Trigger:** SIP-based answer rate (SIP 200) looks healthy, but contact/RPC/conversion rates are poor.
- **Hypotheses:** high AMD/voicemail rate inflating the "answered" count without live engagement; calls answered but immediately hung up (possible spam-forewarning on the recipient's device); poor list/offer fit.
- **Diagnostic checks:** compare AMD rate, contact rate, and average talk time on "answered" calls. A high answer rate with very short average talk time and low AMD rate specifically (i.e., humans answering and immediately hanging up) can indicate the recipient's phone displayed a spam warning — cross-check attestation level and recent labeling signals ([05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md)).
- **Likely causes & actions:** distinguish a list/offer problem (fixable via targeting/script) from a labeling problem (fixable via deliverability remediation) — they require different owners.

---

## Workflow 7: New Number Pool Underperforming Post-Launch

- **Trigger:** a newly deployed number pool shows lower connect/answer rates than the established pool it's supplementing or replacing.
- **Diagnostic checks:** verify warm-up was actually followed (dates, volume ramp) before attributing underperformance to anything else. New numbers without established call history and without STIR/SHAKEN full attestation history are inherently more likely to be scrutinized by carrier analytics regardless of behavior — factor in a reasonable ramp period before judging the pool's steady-state performance.
- **Likely causes & actions:** if warm-up was skipped or rushed, that's the most common and directly fixable cause.

---

## Workflow 8: DNC/Compliance Complaint Received

- **Trigger:** an external complaint (consumer, regulator, carrier) about a specific call or number.
- **Data to pull:** the specific CDR(s) for the number/date/time in question, plus (outside CDR scope) the consent/DNC-scrub record for that number at the time of the call.
- **Diagnostic checks:** run the full compliance-check workflow ([07-compliance-and-legal-considerations.md](07-compliance-and-legal-considerations.md)) against that specific call: calling-hours check, DNC/consent context if available, RND recency if available.
- **Escalation:** complaint-driven investigations should always be escalated to the compliance/legal owner regardless of what the CDR-level check finds — CDR data can inform but not resolve a consent/legal determination on its own.
