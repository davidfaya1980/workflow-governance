# Source artifacts: requirements and specifications

First-draft requirements and specifications for the system this project will build, the **Governance Intake Platform**. The artifacts are the source; the Markdown exports in this folder are the copies we work from.

| Artifact | Type | Version | Original | Local copy |
|---|---|---|---|---|
| Governance Intake Platform: Executive Summary | Slide deck (12 slides) | Draft v0.1, Nov 2026 | [Open artifact](https://claude.ai/code/artifact/9d74035d-5ebf-4abd-ab3c-720745a49b3a) | [governance-intake-platform-exec-summary.md](governance-intake-platform-exec-summary.md) |
| _Second artifact (to be added)_ | | | | |

The deck summarizes a longer design document that is not yet in this repo. It holds the full domain packs, the province-by-province matrix and the full metric set.

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
