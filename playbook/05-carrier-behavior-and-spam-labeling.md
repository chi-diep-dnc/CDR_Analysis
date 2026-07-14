# 05 — Carrier Behavior & Spam-Labeling Risk

## Scope and confidence level

This section covers well-established, publicly-documented mechanisms (STIR/SHAKEN, the general concept of analytics-based call labeling). It deliberately does **not** assert specific carrier or analytics-vendor scoring algorithms as fact, since those are proprietary, change over time, and vary by vendor — treat any claim about a specific vendor's exact behavior as something to verify (via the DNCScrub compliance tools or targeted research) before acting on it, not something to assume from general telecom knowledge.

## STIR/SHAKEN attestation

STIR/SHAKEN is the FCC-mandated framework for authenticating caller ID on IP-based call paths, intended to combat spoofing. Each call is signed by the originating carrier with an attestation level:

| Level | Name | Meaning | Spam-labeling risk implication |
|---|---|---|---|
| **A** | Full attestation | The signing carrier knows the caller, and the caller is authorized to use the calling number | Lowest risk — this is the target state for legitimate outbound campaigns |
| **B** | Partial attestation | The signing carrier knows the caller (has a customer relationship) but cannot verify authorization to use that specific number | Moderate risk — common for call centers/BPOs using numbers not directly assigned to them |
| **C** | Gateway attestation | The signing carrier has no relationship with the caller (e.g., call entered via an international gateway) and cannot verify anything | Highest risk — strongly associated with spam/robocall labeling |

**Implication for analysis:** if `stir_shaken_attestation` is available in the CDR (see [02-canonical-data-schema.md](02-canonical-data-schema.md)), a shift in the attestation-level mix for a number pool is a leading indicator worth correlating against spam-labeling incidents and connect-rate trends.

## What drives negative call labeling (general mechanisms)

Carrier and third-party call-analytics systems generally weigh some combination of the following signals — the specific weighting and thresholds are proprietary and vendor-specific, so treat the list as "things to check," not a scoring formula:

- **Call volume spikes** relative to a number's own recent history (see `knowledge_base.json` → `call_deliverability_guide.call_volume_consistency` for the concrete ±70–100% of 14-day-average guidance already captured).
- **Low answer-to-completion engagement** — very short average call durations across many calls from one number can look like robocalling behavior even when it isn't.
- **Sequential/adjacent number dialing patterns** — dialing many numbers in numeric sequence from one campaign is a classic robocall signature that detection systems watch for.
- **New, unwarmed numbers** suddenly generating high volume (see `knowledge_base.json` → `call_deliverability_guide.warm_up_new_numbers`).
- **Consumer complaint reports** filed directly with carriers, the FCC, or the FTC against a specific number.
- **STIR/SHAKEN attestation level**, as above — lower attestation correlates with higher scrutiny.
- **Caller ID reuse/sharing across unrelated campaigns or businesses** (see `knowledge_base.json` → `call_deliverability_guide.caller_id` rules against sharing a caller ID across multiple sellers/brands).

## Diagnostic signal table: CDR patterns → likely carrier-behavior interpretation

Use this to generate hypotheses during an investigation ([10-investigation-workflows.md](10-investigation-workflows.md)), not as a definitive diagnosis — always corroborate with volume/timing data before concluding a carrier is actively blocking calls.

| Observed CDR pattern | Plausible interpretation(s) | What to check next |
|---|---|---|
| Spike in SIP 608 (Rejected/Spam-like) | The number pool is being actively flagged as spam by carrier/analytics providers | Check recent volume-consistency and warm-up compliance for the affected numbers; check for recent number-pool changes |
| Spike in SIP 503 (Service Unavailable) | Could be genuine network congestion/maintenance, OR a carrier using 503 as a soft block for spam mitigation | Check whether the spike is carrier-specific or destination-region-specific (genuine congestion tends to cluster by carrier/region; targeted blocking tends to cluster by originating number) |
| Spike in SIP 403 (Forbidden) | The destination or a downstream carrier is refusing the call outright | Distinguish "many different destinations refusing" (points to originating-number reputation) from "one destination/carrier refusing everything" (points to a specific carrier relationship issue) |
| Rising trend in AMD-detected/short-duration connects with falling RPC rate | Numbers may be getting through but engagement quality is degrading — possible early labeling before outright blocking | Cross-check against attestation level trend and volume-consistency compliance |
| Sudden drop in connect rate isolated to one number pool, other pools unaffected | Points to that specific pool's reputation/labeling, not a systemic issue | Check that pool's recent warm-up and volume history first |

## Robocall Mitigation Database (RMD) context

Under FCC rules, voice service providers must generally either fully implement STIR/SHAKEN or file in the Robocall Mitigation Database attesting to their call-blocking/mitigation practices. A caller's downstream deliverability can be affected by their carrier's own RMD standing, not just the caller's own behavior — this is a variable largely outside the dialing operation's direct control, but worth ruling in/out when a deliverability problem doesn't correlate with anything on the calling side.
