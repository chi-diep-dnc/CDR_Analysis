# 11 — Glossary

| Term | Definition |
|---|---|
| **CDR** | Call Detail Record — a single log entry describing one call attempt (see [01-cdr-fundamentals.md](01-cdr-fundamentals.md)). |
| **ANI** | Automatic Number Identification — the originating number/caller ID transmitted with a call. |
| **DNIS** | Dialed Number Identification Service — the destination number that was dialed. |
| **NPA-NXX** | The area code (NPA) + exchange (NXX) prefix of a phone number — often a more reliable basis for timezone resolution than a mailing ZIP code, since it reflects the number's assigned rate center. |
| **ZIP3** | The first 3 digits of a 5-digit ZIP code, used as a coarse geographic grouping. Several states have timezone boundaries that split a single ZIP3 prefix — see `knowledge_base.json` → `timezone_reference` `zip3_reliable` flags. |
| **SIP** | Session Initiation Protocol — the signaling protocol used to set up, manage, and tear down most modern VoIP/carrier calls; its numeric response codes are covered in [03-sip-code-reference.md](03-sip-code-reference.md). |
| **AMD** | Answering Machine Detection — automated technology that attempts to distinguish a human answer from a machine/voicemail answer. |
| **STIR/SHAKEN** | The FCC-mandated caller ID authentication framework for IP call paths, using cryptographic attestation levels A/B/C — see [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md). |
| **Attestation (A/B/C)** | The STIR/SHAKEN confidence level a carrier assigns to a call's caller-ID legitimacy: A = full, B = partial, C = gateway/none. |
| **RMD** | Robocall Mitigation Database — the FCC database where voice service providers must file their STIR/SHAKEN implementation or call-blocking/mitigation practices. |
| **RND** | Reassigned Numbers Database — the FCC-designated resource for checking whether a phone number has been reassigned to a new subscriber since a prior consent was obtained. |
| **DNC** | Do Not Call — refers to both the National DNC Registry and a business's own internal do-not-call list. |
| **EBR** | Established Business Relationship — a recognized (though not universal — see Indiana exception) exemption allowing calls to DNC-listed numbers within defined time windows (18 months post-purchase, 3 months post-inquiry). |
| **PEWC** | Prior Express Written Consent — required for prerecorded/AI-voice telemarketing calls and for calling DNC-listed numbers absent a valid EBR. |
| **TCPA** | Telephone Consumer Protection Act — the primary U.S. federal law governing telemarketing calls, texts, and consent requirements. |
| **Connect Rate** | Share of dial attempts where `connect_status == "Connected"` — see `knowledge_base.json` → `metrics_definitions.connect_rate`. |
| **Answer Rate** | Share of dial attempts receiving SIP 200 — network-level acceptance only, not proof of human contact (see [03-sip-code-reference.md](03-sip-code-reference.md)). |
| **Right Party Contact (RPC)** | A connected call confirmed to have reached the correct/intended person, as distinct from any human contact (which may include wrong numbers). |
| **Abandonment Rate** | Share of connected calls where no agent was available (queue/system drop) — a compliance-relevant metric with a 3%/30-day threshold. |
| **Predictive Dialer** | An outbound dialing system that paces call volume based on predicted agent availability, which is also the mechanism most directly responsible for abandonment-rate risk if miscalibrated. |
| **Spam Likely / Spam-Labeling** | A designation applied by a carrier or third-party analytics provider marking a caller ID as likely unwanted/fraudulent to the called party's device — see [05-carrier-behavior-and-spam-labeling.md](05-carrier-behavior-and-spam-labeling.md). |
| **Branded Calling** | A caller-ID enhancement mechanism (distinct from basic Caller ID) that displays a verified business name/logo/reason-for-calling to the called party, generally correlated with higher attestation/trust. |
