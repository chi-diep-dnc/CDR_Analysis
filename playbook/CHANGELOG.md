# Playbook Changelog

## v1.0 — 2026-07-14

Initial playbook created, built on top of the existing `knowledge_base.json` (SIP codes, dialer statuses, timezone reference, DNC.com deliverability/compliance guide, established in the same session). Established the two-layer structure (JSON = data, `playbook/*.md` = knowledge/methodology) and the maintenance rules in `README.md`.

Sections added:
- `01-cdr-fundamentals.md` — CDR concept, call lifecycle, platform-diversity problem
- `02-canonical-data-schema.md` — normalized field schema + platform mapping procedure
- `03-sip-code-reference.md` — SIP interpretation methodology, class-level fallback rules
- `04-dialer-status-taxonomy.md` — canonical status categories + cross-platform mapping method
- `05-carrier-behavior-and-spam-labeling.md` — STIR/SHAKEN, spam-labeling drivers, diagnostic signal table
- `06-deliverability-best-practices.md` — actionable checklist derived from the compliance guide
- `07-compliance-and-legal-considerations.md` — compliance-check workflow, CDR-alone scope boundary
- `08-kpi-catalog.md` — full KPI formula catalog
- `09-analytical-methodologies.md` — funnel, time-series, segmentation, root-cause, anomaly-detection methods
- `10-investigation-workflows.md` — 8 structured investigation runbooks
- `11-glossary.md` — term definitions

**Known gaps to fill as data becomes available:** platform-specific field mappings beyond the one dialer platform already captured in `knowledge_base.json`; real STIR/SHAKEN attestation data once available in CDR exports; validated ZIP5/county-level timezone data for the states currently flagged `zip3_reliable: false`.
