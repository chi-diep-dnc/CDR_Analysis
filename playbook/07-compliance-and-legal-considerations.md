# 07 — Compliance & Legal Considerations

**Disclaimer, inherited from the source guide:** everything in `knowledge_base.json` → `call_deliverability_guide` and referenced here is guidance, not legal advice. Recommend legal counsel review for actual outbound-calling compliance programs. This playbook section describes how to *check CDR data against* known rules — it is not itself a compliance certification.

## Compliance-check workflow for a batch of CDRs

For any set of outbound call records under review, run these checks in order:

1. **Resolve destination timezone.**
   - Prefer NPA-NXX (area code + exchange) based timezone resolution when available — it maps to the phone number's actual rate center.
   - Otherwise, derive `destination_state` from `destination_zip`, then look up `knowledge_base.json` → `timezone_reference.states`.
   - **Check the `split` and `zip3_reliable` flags.** For the 15 states flagged `zip3_reliable: false` (Alaska, Florida, Idaho, Indiana, Kansas, Kentucky, Michigan, Nebraska, North Dakota, South Dakota, Tennessee — plus Arizona/Nevada/Oregon/Texas which are split but ZIP3-reliable), ZIP3 alone cannot be trusted at the boundary — see each entry's `split_notes` for the county-level detail needed, and treat any resulting timezone determination for boundary-adjacent ZIPs as lower-confidence.
   - Also check the `dst_rule` field for whether the resolved timezone observes DST at the call's date (Arizona outside Navajo Nation and Hawaii do not).

2. **Check the call timestamp against legal calling hours.**
   - Federal floor: 8:00 AM–9:00 PM local time (`call_deliverability_guide.calling_time_restrictions.telemarketing`).
   - State-specific window: look up the destination state in `call_deliverability_guide.calling_time_restrictions.state_specific_windows.states` — many states have narrower windows, some distinguish live-operator vs. prerecorded-message calls, some prohibit Sunday calling entirely.
   - Flag any call placed outside the applicable window as a compliance violation candidate.

3. **Check DNC/consent status context** (requires DNC-scrub metadata alongside the CDR, not derivable from the CDR alone):
   - Was the number scrubbed against the National DNC Registry within the last 31 days? (`do_not_call_compliance`)
   - If the number is on the DNC list, is there a valid PEWC or EBR on file? Note state-specific exceptions — e.g., Indiana does not recognize EBR as a valid exemption; PEWC is required there.
   - EBR validity windows: 18 months from last purchase (transactional), 3 months from last inquiry.

4. **Check reassigned-number (RND) status** (also requires external RND data, not derivable from the CDR alone):
   - Was the number checked against current RND data before calling, on a 30–45 day cadence? (`reassigned_number_scrubbing_compliance`)
   - A DNC-listed number that was recently reassigned and called anyway is a violation regardless of prior PEWC.

5. **Check abandonment rate compliance** — see [06-deliverability-best-practices.md](06-deliverability-best-practices.md) for the ≤3%/30-day rule; this is simultaneously a compliance and deliverability check.

6. **Check disclosure-relevant call types**, if the analysis has access to call content/recording metadata (not from CDR alone): prerecorded/AI-voice calls require PEWC and specific opening disclosures (`ai_artificial_voice`, `oral_disclosures.prerecorded_and_artificial_ai_voice`) — flag any AI/prerecorded-flagged calls lacking a linked consent record.

## What CDR data alone can and cannot tell you

Be explicit about this boundary in any compliance analysis:

- **CDR alone CAN determine:** call timestamp vs. legal hours (once timezone is resolved), abandonment rate, ring time, volume patterns.
- **CDR alone CANNOT determine:** DNC scrub recency, PEWC/EBR status, RND scrub recency, or disclosure content — these require joining the CDR against separate compliance/consent records. Never assume compliance on these dimensions just because the CDR itself looks clean; say explicitly when a compliance question is out of scope for CDR-only data.

## Reporting compliance findings

When a potential violation is found, report: the specific rule violated (cite the `knowledge_base.json` field, e.g. `calling_time_restrictions.state_specific_windows` for TN), the affected records (count, date range, number pool), and the confidence level of the underlying timezone/geography resolution (per the `zip3_reliable` caveat above) — a finding built on a low-confidence ZIP3 timezone guess should be labeled as such, not presented with the same confidence as one built on NPA-NXX resolution.
