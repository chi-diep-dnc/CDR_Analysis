# 02 — Canonical Data Schema

Before computing any KPI or running any investigation workflow, normalize the source platform's raw export into this canonical schema. This is what makes analysis comparable across platforms and what every formula in [08-kpi-catalog.md](08-kpi-catalog.md) is written against.

## Canonical fields

| Canonical field | Type | Description | Typical raw-export aliases to look for |
|---|---|---|---|
| `call_id` | string | Unique identifier for this call attempt | `CallID`, `call_uuid`, `session_id`, `record_id` |
| `campaign_id` | string | Campaign/list this call belongs to | `campaign`, `list_id`, `CampaignName` |
| `timestamp_start` | datetime (UTC + source tz) | When the call attempt began | `call_date`, `start_time`, `OriginationTime` |
| `timestamp_answer` | datetime, nullable | When the call was answered (if it was) | `answer_time`, `connect_time` |
| `timestamp_end` | datetime | When the call ended | `end_time`, `hangup_time`, `TerminationTime` |
| `duration_total_sec` | integer | Total call length, dial to hangup | `length_in_sec`, `call_length`, `duration` |
| `duration_talk_sec` | integer, nullable | Talk time after answer (excludes ring time) | `talk_time`, `billsec` |
| `ani` | string (E.164 or 10-digit) | Originating number / caller ID used | `caller_id`, `CallerIDNum`, `from_number` |
| `dnis` | string (E.164 or 10-digit) | Destination number dialed | `phone_number`, `lead_phone`, `to_number` |
| `direction` | enum | `outbound` or `inbound` | `call_type`, `direction` |
| `sip_response_code` | integer, nullable | Final SIP response for this leg, if exposed — see [03-sip-code-reference.md](03-sip-code-reference.md) | `sip_code`, `hangup_cause_code`, `q850_cause` |
| `dialer_disposition_raw` | string | The platform's own status/disposition string, verbatim | `disposition`, `status`, `hangup_cause`, `wrapup_code` |
| `dialer_disposition_canonical` | enum | `dialer_disposition_raw` mapped into the taxonomy in [04-dialer-status-taxonomy.md](04-dialer-status-taxonomy.md) | *(derived, not sourced)* |
| `connect_status` | enum: `Connected` / `Not Connected` | Derived from `dialer_disposition_canonical` (see taxonomy doc) — the basis for connect/disconnect rate | *(derived, not sourced)* |
| `agent_id` | string, nullable | Agent who handled the call, if any | `agent`, `AgentID`, `user_id` |
| `transfer_flag` | boolean | Whether the call was transferred (to a buyer, closer, etc.) | `transferred`, `xfer` |
| `destination_zip` | string, nullable | ZIP code of the destination number's registered location, if known | `zip`, `postal_code` |
| `destination_state` | string, nullable | Derived from `destination_zip` via a ZIP→state lookup, or sourced directly | `state`, `st` |
| `carrier` | string, nullable | Terminating/originating carrier, if exposed | `carrier_name`, `otn` |
| `stir_shaken_attestation` | enum: `A`/`B`/`C`, nullable | Attestation level, if exposed — see [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md) | `attestation`, `shaken_level` |

## Derived fields — how to compute them

- **`dialer_disposition_canonical` and `connect_status`**: look up `dialer_disposition_raw` against the platform-specific mapping you've built (see [04-dialer-status-taxonomy.md](04-dialer-status-taxonomy.md)). For the one platform already captured in `knowledge_base.json` → `dialer_status`, this is a direct lookup by code; `connect_status` is already a field on each entry there.
- **`destination_state`**: if not present directly, derive from `destination_zip`'s first 3 digits (ZIP3) or full ZIP via a ZIP→state reference, then use `knowledge_base.json` → `timezone_reference.states` to get the timezone. Watch for the 15 split states flagged with `zip3_reliable: false` — for those, ZIP3 alone is not sufficient (see [07-compliance-and-legal-considerations.md](07-compliance-and-legal-considerations.md)).

## Normalization procedure for a new/unfamiliar platform export

1. **Inventory the raw fields.** List every column/field in the export with a few sample values.
2. **Identify the timestamp fields.** Determine whether the export has one timestamp or several (start/answer/end) and what timezone they're in (often UTC or the dialer's server timezone — confirm, don't assume).
3. **Identify the status/disposition field(s).** Some platforms split this into a hangup cause (network-level) and an agent disposition (business-level) — capture both if present, as they answer different questions.
4. **Identify the SIP/carrier response field, if any.** Many predictive dialers do not expose this at all — that's expected, not an error. Do not fabricate a SIP code when none is present; leave `sip_response_code` null and rely on `dialer_disposition_canonical` for connect/not-connected classification instead.
5. **Identify the geography field(s).** ZIP, state, or area-code-only — note which, since precision affects how confidently you can resolve timezone (area code / NPA-NXX is generally more reliable than ZIP for timezone, since it maps to the actual phone number's rate center rather than a mailing address; prefer NPA-NXX-based timezone resolution when both are available).
6. **Map every distinct raw disposition value seen in the data to a canonical category** (see taxonomy doc) before computing any rate. Do not guess a mapping for a status you don't understand — flag it as unmapped and ask, rather than defaulting it to a category that could silently distort connect/abandonment rates.
7. **Record the mapping.** Once built, save the raw-to-canonical mapping for that platform so it doesn't need to be rebuilt on the next dataset from the same source.
