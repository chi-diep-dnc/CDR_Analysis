# Telecom CDR Analysis Playbook

**Purpose:** This is the authoritative knowledge base for analyzing Call Detail Records (CDRs) across dialing platforms — SIP interpretation, dialer status mapping, carrier behavior, deliverability, compliance, KPIs, and investigation workflows. It is written for an AI agent (or human analyst) to consistently and accurately interpret, normalize, and analyze CDR data regardless of source platform.

## How this playbook is organized

Two layers, kept deliberately separate:

- **`../knowledge_base.json`** — the structured data layer: exact SIP code tables, dialer status tables, timezone reference, and the DNC.com deliverability/compliance guide. This is where exhaustive lookups live. Treat it as the single source of truth for facts — if a playbook section and the JSON ever disagree, the JSON wins (and the playbook section is stale and should be fixed).
- **`playbook/*.md`** (this directory) — the knowledge layer: narrative domain knowledge, normalization methodology, KPI formulas, analytical methods, and investigation runbooks. This is where reasoning and process live.

An agent using this playbook should: read the relevant `.md` section(s) for methodology and interpretation, then query `knowledge_base.json` for the exact codes/thresholds/values needed to execute that methodology on real data.

## Table of contents

| # | File | Covers |
|---|------|--------|
| 01 | [cdr-fundamentals.md](01-cdr-fundamentals.md) | What a CDR is, call lifecycle, why platforms differ |
| 02 | [canonical-data-schema.md](02-canonical-data-schema.md) | The normalized field schema every platform's export should be mapped into |
| 03 | [sip-code-reference.md](03-sip-code-reference.md) | SIP code interpretation methodology + pointer to the JSON table |
| 04 | [dialer-status-taxonomy.md](04-dialer-status-taxonomy.md) | Canonical status categories + cross-platform mapping method |
| 05 | [carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md) | STIR/SHAKEN, spam-labeling drivers, carrier-side blocking signals |
| 06 | [deliverability-best-practices.md](06-deliverability-best-practices.md) | Actionable checklist derived from the compliance guide |
| 07 | [compliance-and-legal-considerations.md](07-compliance-and-legal-considerations.md) | DNC/TCPA/timezone compliance-check workflow |
| 08 | [kpi-catalog.md](08-kpi-catalog.md) | Every KPI: formula, required fields, interpretation, pitfalls |
| 09 | [analytical-methodologies.md](09-analytical-methodologies.md) | Funnel analysis, segmentation, anomaly detection, root-cause decomposition |
| 10 | [investigation-workflows.md](10-investigation-workflows.md) | Step-by-step runbooks for common investigation triggers |
| 11 | [glossary.md](11-glossary.md) | Term definitions |
| — | [CHANGELOG.md](CHANGELOG.md) | Playbook version history |

## Maintenance rules (read before editing)

1. **Data vs. knowledge.** A new SIP code, dialer status, state law, or timezone fact goes into `knowledge_base.json`, not into a playbook `.md` file. A new methodology, KPI, or investigation pattern goes into the playbook. Don't duplicate JSON table contents into markdown prose — link to the JSON section by name instead.
2. **Verify before writing.** Per project standard, do not add specific factual claims (legal citations, geographic boundaries, thresholds) from memory alone — verify via the DNCScrub compliance tools or research, and flag anything unverified explicitly rather than presenting it as certain.
3. **Platform-neutral by default.** The dialer status table in `knowledge_base.json` was captured from one specific platform's disposition codes. Playbook sections must describe the *canonical taxonomy* categories, not that platform's exact code strings, so the playbook stays useful when a new platform's export uses different vocabulary.
4. **Every new section gets a changelog entry.** Log what changed and why in `CHANGELOG.md` so future readers understand the playbook's evolution.
5. **No silent contradictions.** If new knowledge conflicts with something already written, update the existing section in place (with a changelog note) rather than adding a second, conflicting statement elsewhere.
