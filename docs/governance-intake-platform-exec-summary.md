# Governance Intake Platform: Executive Summary

> Exported from the "Governance Intake Platform: Executive Summary" slide deck (Draft v0.1, November 2026) for use as implementation-planning context. Slide content is followed by the slide's speaker notes, marked **Notes**. Note: the deck summarizes a longer design document (referenced throughout) that is not included here.

**Deck sections**
1. Why governance intake needs to change (slides 1–2)
2. How the platform works and what it covers (slides 3–5)
3. A worked example and the safeguards that make it trustworthy (slides 6–9)
4. Targets, roadmap and the decisions needed to start (slides 10–12)

---

## 1. Governance Intake Platform

*One front door from idea to launch and run state*

Executive summary | Draft v0.1 | November 2026

**Notes:** Executive summary of the Governance Intake Platform design document (draft v0.1). The ask at the end is four decisions to start discovery.

---

## 2. Governance needs surface late, one function at a time

- **Found too late:** Teams learn which reviews apply after design or build has started, when changes cost the most.
- **Asked many times:** Each function runs its own intake form, so requesters answer overlapping questions again and again.
- **No single record:** Nobody can show what applied, who approved, or which obligations continue after launch.

**The result:** rework, launch delays, and gaps that surface in audits rather than at intake.

**Notes:** Baseline metrics (cycle time, late reviews, missed-review incidents, SME hours) will be collected in discovery to size the problem.

---

## 3. The agent gathers facts, rules decide, people approve

Flow:

1. **Intake agent** — Reads the brief and asks only the questions that matter
2. **Shared facts** — One taxonomy; every fact asked once and confirmed
3. **Rules engine** *(highlighted)* — Owner-maintained, versioned rules decide which domains apply
4. **Reviews and approvals** — Routed to named people, in parallel, with SLAs
5. **Run state** — Recurring obligations; changes re-open review

**Design principle:** the AI never decides. Deterministic, versioned rules decide what applies, so every requirement traces back to a rule and the facts that triggered it.

**Notes:** Unknown answers default to triggering review, so missing information produces over-review rather than a silent miss.

---

## 4. Twelve domains plug in through rules their owners control

| Domain | Typical trigger |
|---|---|
| Privacy | Personal or sensitive data |
| Information Security | Confidential data, new code |
| Legal | Contracts, offers, public claims |
| Procurement and Vendors | Any vendor; spend of $25k or more |
| AI Governance | Any AI use; decisions on people |
| Finance | Spend, pricing, tax, accounting |
| Records | New systems of record |
| Regulatory and Compliance | Licensing, filings, accessibility |
| HR and People | Monitoring, reorgs, hiring tools |
| Risk and Resilience | Critical services, risk appetite |
| Data Governance | New data, sharing, data products |
| Enterprise Architecture | New tech, core platform changes |

**Notes:** Each domain pack defines its own attributes, applicability rules, policies, approval chains by tier, and run-state obligations. Regulatory and Compliance now includes accessibility, French-language requirements, licensing and regulatory filings, sanctions and anti-bribery. The newer packs, Enterprise Risk and Operational Resilience, Data Governance and Enterprise Architecture, cover areas nearly every large company already reviews. The packs in the design doc are proposed defaults for each owner to accept or edit.

---

## 5. Ten provinces in scope, with federal law as the baseline

**10 provinces covered by rules.** Federal baseline in every province: PIPEDA, CASL, the Competition Act and, for federally regulated businesses, federal labour, pay equity and accessibility law.

| Province | Province-specific rules |
|---|---|
| Quebec | Law 25 privacy impact and transfer assessments, automated-decision notice; French-language Charter |
| Ontario | AODA accessibility standards; monitoring-policy and AI-in-hiring disclosure rules |
| Alberta and BC | Their own private-sector privacy acts (PIPA) apply instead of PIPEDA in-province |
| Manitoba | Accessibility standards that apply to private organizations |

Territories and foreign jurisdictions route to manual Legal review. Owners verify every provincial reference before go-live.

**Notes:** The design doc has a full province-by-province matrix (privacy, health information, accessibility, pay equity and transparency, other flags) with sources as of November 2026. Watch items: federal Bill C-36 and Ontario's Consumer Protection Act, 2023, neither yet in force.

---

## 6. One AI chatbot project triggers ten of twelve domains

**Example project: Smart Support Assistant**
- Generative-AI chatbot answering billing questions from customer account history
- Third-party model, new vendor, $180k in year one
- Users in Ontario, Quebec and Alberta; vendor hosts outside Canada

**Resulting domain triggers by tier**

| Tier | Domains |
|---|---|
| Tier 3: escalate | Privacy, Procurement |
| Tier 2: expert review | Security, AI Governance, Legal, Regulatory and Compliance, Records, Architecture |
| Tier 1: notify | Finance, Data Governance |
| Not triggered | HR and People, Risk and Resilience |

Run state set up automatically: annual PIA refresh, monthly AI monitoring, annual vendor reassessment, transcript deletion, accessibility scans, renewal alerts.

**Notes:** HR is not triggered because the project does not affect employees. The agent asks one follow-up (will support agents' work or monitoring change?), which would add HR review if yes. Enterprise Risk and Resilience is not triggered unless billing support is on the critical services list; the agent asks about that too.

---

## 7. Built to stand up to auditors, regulators and experts

- **Deterministic rules:** Same facts and rule versions always give the same result, and every trigger is explained.
- **Conservative defaults:** Unknown or ambiguous answers trigger review. Missing information never causes a silent miss.
- **People make the decisions:** The agent cannot approve, waive requirements or change outcomes. Named approvers do.
- **Audit-ready by design:** Tamper-evident log, one-click evidence packs, and privilege controls for legal advice.

**Notes:** The platform itself goes through its own intake: it holds sensitive plans (M&A, reorganizations, legal advice), so it must meet its own Tier 3 bar, with Canadian data residency by default.

---

## 8. Reporting at two levels, run from one admin dashboard

**Global reporting** — for governance, executives and audit
- Portfolio by stage, risk tier and province
- Cycle times and bottlenecks
- Exceptions and overdue obligations
- Evidence packs and regulatory inventories

**Per-domain reporting** — for each domain owner
- Review queue and SLA status
- How often each rule fires
- Waivers, exceptions and obligations due
- Rule health and reviewer workload

**Admin dashboard** *(highlighted)* — for the governance team
- Tracker of every project, stage and next gate
- Status set automatically: on track, at risk, blocked
- Alerts, reassignment and escalation
- Every admin action logged

**Notes:** All figures are computed from project records, rule evaluations and the audit log, never maintained by hand. Confidential projects count in totals but are hidden from lists unless the viewer has access. Admins cannot approve on a domain's behalf or waive requirements. The MVP includes the project tracker, computed status, alerts and per-domain queues; the full report sets and scheduled digests follow in the Scale phase.

---

## 9. Four capabilities cut repeat work and keep approvals current

- **Reuse earlier approvals:** Approved assessments of a shared vendor, platform or AI model carry over to new projects, with an expiry date. Only facts that changed go to review.
- **Reviewer assistant:** Briefs each specialist, drafts assessments such as the PIA and threat model from confirmed facts, and flags what's unusual. A named reviewer decides.
- **Regulatory change tracking:** New laws and guidance go to each owner's rule review queue, with the live projects they would affect and a due date tied to the effective date.
- **Annual attestations:** Once a year, project owners confirm the facts behind their approvals still hold. A changed answer re-opens review for that part only.

**Notes:** Approval reuse is often the biggest time saver in large companies: owners decide which requirements can be reused and for how long, and conditions on the original approval carry over. The reviewer assistant follows the same guardrails as the intake agent: it cannot approve, reject or waive. Example regulatory watch items today: federal Bill C-36 and Ontario's Consumer Protection Act, 2023. Phasing: attestations, regulatory tracking and vendor reuse in the Scale phase; reviewer assistant and wider reuse in Optimize.

---

## 10. What success looks like after 12 months

| Target | Metric |
|---|---|
| **90%+** | of in-scope projects enter through intake |
| **30%** | faster from kickoff to all approvals |
| **0** | missed-review incidents in pilot scope |
| **40%** | less expert time spent on triage |

Proposed targets. Final values are set once discovery establishes today's baselines.

**Notes:** Further metrics in the design doc: median questions per intake under 25, agent extraction accuracy, requester satisfaction of 4 out of 5 or higher, and 100% of rules reviewed within 12 months.

---

## 11. Pilot four domains by mid-2027, then scale to all twelve

| Phase | When | Scope |
|---|---|---|
| **Discovery** | Dec 2026 – Feb 2027 | Expert and requester interviews; shared taxonomy v1; back-test 20 past projects; build vs buy decision |
| **MVP pilot** | Mar – Jun 2027 | Privacy, Security, Vendor Risk, AI Governance; intake agent and rules engine; admin dashboard and tracker; one business unit |
| **Scale** | Jul – Dec 2027 | All twelve domains; run-state obligations; work-management and procurement links; attestations, regulatory tracking, vendor approval reuse |
| **Optimize** | 2028 | Reviewer assistant; wider approval reuse; evidence automation; portfolio analytics; multi-entity rules |

**Gates**
- **Gate 1:** taxonomy and back-test signed off (Discovery → MVP pilot)
- **Gate 2:** pilot targets met (MVP pilot → Scale)
- **Gate 3:** all owners approve their packs (Scale → Optimize)

**Notes:** Dates are planning assumptions. Each phase starts only when the gate before it is met.

---

## 12. Four decisions to start discovery

1. Confirm an executive sponsor
2. Name the owner of the shared fact taxonomy
3. Choose the pilot business unit
4. Decide whether intake is mandatory for all projects or above a size threshold

**Notes:** Build vs buy is assessed during discovery and comes back for decision at the first gate.
