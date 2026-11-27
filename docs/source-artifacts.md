# Source artifacts: requirements and specifications

First-draft requirements and specifications for the system this project will build, the **Governance Intake Platform**. The artifacts are the source; the Markdown exports in this folder are the copies we work from.

| Artifact | Type | Version | Original | Local copy |
|---|---|---|---|---|
| Governance Intake Platform: Executive Summary | Slide deck (12 slides) | Draft v0.1, Nov 2026 | [Open artifact](https://claude.ai/code/artifact/9d74035d-5ebf-4abd-ab3c-720745a49b3a) | [governance-intake-platform-exec-summary.md](governance-intake-platform-exec-summary.md) |
| Governance Intake Platform: Design Document | Doc | Draft v0.1, Nov 24, 2026 | [Open artifact](https://claude.ai/code/artifact/9e1c0796-d0b3-4597-96c2-cd668dcc27fd) | [governance-intake-platform-design-doc.md](governance-intake-platform-design-doc.md) (no diagrams) |

The deck summarizes the design document. **The design document is the more detailed source**: where the two differ, follow the design document and note the difference.

## Executive summary at a glance

**Problem.** Governance reviews surface late, each function runs its own intake form, and there is no single record of what applied, who approved, or which obligations continue after launch.

**How it works.** An intake agent gathers facts, a shared taxonomy holds them, a deterministic versioned rules engine decides which domains apply, reviews are routed in parallel to named approvers with SLAs, and run-state obligations continue after launch. The AI never decides, and unknown answers default to triggering review.

**Scope.**
- 12 domains: Privacy, Information Security, Legal, Procurement and Vendors, AI Governance, Finance, Records, Regulatory and Compliance, HR and People, Risk and Resilience, Data Governance, Enterprise Architecture. Each is a pack owned and maintained by its domain owner.
- Review tiers: Tier 1 notify, Tier 2 expert review, Tier 3 escalate.
- 10 Canadian provinces on a federal baseline (PIPEDA, CASL, Competition Act). Territories and foreign jurisdictions go to manual Legal review.

**Safeguards.** Deterministic, explainable rules; conservative defaults; people make every decision; tamper-evident audit log, evidence packs and legal-privilege controls. The platform must pass its own Tier 3 bar, with Canadian data residency by default.

**Reporting.** Global reporting, per-domain reporting, and an admin dashboard with a project tracker, computed status and alerts. All figures come from records, never maintained by hand.

**Later capabilities.** Approval reuse, reviewer assistant, regulatory change tracking, annual attestations.

**12-month targets (proposed).** 90%+ of in-scope projects through intake; 30% faster to all approvals; 0 missed reviews in pilot scope; 40% less expert triage time.

**Roadmap.**

| Phase | When | Gate to exit |
|---|---|---|
| Discovery | Dec 2026 – Feb 2027 | Gate 1: taxonomy and back-test signed off |
| MVP pilot (Privacy, Security, Vendor Risk, AI Governance; one business unit) | Mar – Jun 2027 | Gate 2: pilot targets met |
| Scale (all 12 domains) | Jul – Dec 2027 | Gate 3: all owners approve their packs |
| Optimize | 2028 | |

**Open decisions to start discovery.**
1. Confirm an executive sponsor
2. Name the owner of the shared fact taxonomy
3. Choose the pilot business unit
4. Decide whether intake is mandatory for all projects or only above a size threshold

Build vs buy is assessed in Discovery and decided at Gate 1.

## Design document: where to find things

| Topic | Section in the design doc |
|---|---|
| Goals, non-goals, personas | Problem, goals and non-goals; Users and personas |
| Data model (project, attribute, domain, rule, requirement, approval chain, obligation, evidence, exception) | Core concepts and domain model |
| Shared attribute taxonomy (`proj.*`, `geo.*`, `data.*`, `tech.*`, `ai.*`, `vendor.*`, `fin.*`, `legal.*`, `ux.*`, `hr.*`, `rec.*`) | Shared project attribute taxonomy |
| Provinces, federal baseline, provincial law matrix | Jurisdiction coverage: Canadian provinces |
| Rule schema, evaluation (fixed point, merge, unknown policy), versioning, conflict checks | Rules engine and rule language |
| Intake agent flow, policy Q&A, guardrails, evaluation set | Intake agent design |
| Rule builder, requirement library, approval chain builder, testing, back-testing, change control | SME authoring experience |
| Gates, SLAs (5 business days Tier 2, 10 Tier 3), delegation, exceptions, re-triggering | Workflow, approvals and lifecycle |
| 12 domain packs (taxonomy, rules, policies, approval chain, run-state obligations) | Domain rule packs 1 to 12 |
| Cross-domain triggers | Cross-domain trigger map |
| Integrations and event stream, by phase | Integrations |
| Platform's own security, privilege, audit log, AI data handling, retention | Security, privacy and privilege of the platform itself |
| Global and per-domain reports, admin dashboard | Reporting, audit and evidence |
| Approval reuse, reviewer assistant, regulatory change tracking, attestations, rule maintenance assistant | Additional capabilities |
| Availability, performance, scale, localization, residency, interoperability | Non-functional requirements |
| Risks and open questions | Risks, mitigations and open questions |
| 12-month metrics | Success metrics |
| Worked example (Smart Support Assistant) | Appendix: worked example |

## Key details the design doc adds

- **Rule IDs** use a domain prefix and number: PRV, SEC, LGL, PRC, AIG, FIN, REC, RC, HR, ERM, DG, EA (e.g., `PRV-02`). Rules are immutable once active; edits create a new version with an effective date, and projects keep a frozen snapshot of rule versions at each gate.
- **Unknown answers** follow each rule's `unknown_policy` (`trigger`, `ask_sme` or `ignore`), defaulting to `trigger`.
- **Gates within a project:** a design gate (PIA, threat model, AIA, vendor selection) and a pre-launch gate (tests, contracts, notices, final sign-offs).
- **Non-functional targets:** 99.9% monthly availability, with the engine degrading to "trigger all Tier 2 reviews", never "none"; rule evaluation under 2 s for 1,000 rules; agent first response under 5 s; 5,000 projects a year, 50 domains, 2,000 rules; English and French; WCAG 2.1 AA; Canadian hosting by default; rules and taxonomy exportable as JSON/YAML.
- **Extra metrics:** reviews started after build under 10%; extraction precision 95%+ and recall 99%+ on triggering attributes; under 25 questions per intake; CSAT 4/5+; 100% of rules reviewed within 12 months.
- **MVP integrations:** SSO, HRIS org chart, Slack/Teams, email, policy repository. Work management, procurement, TPRM/GRC, CLM and data catalog/CMDB come in Phase 2; CI/CD scanners, e-signature and BI in Phase 3.

## Open questions (from the design doc)

- [ ] Who owns the shared attribute taxonomy: central Legal Ops, Risk, or a PMO?
- [ ] Build, buy, or extend an existing GRC platform? (Build-vs-buy analysis to be written.)
- [ ] Which business unit pilots first?
- [ ] What is the threshold for a "material change" that re-triggers review?
- [ ] Is intake mandatory for all projects, or above a size threshold?
- [ ] Final dollar and volume thresholds for each domain's tiers
- [ ] Executive sponsor (from the deck)
