# Governance Intake Platform: Design Document (Draft v0.1)

*Nov 24, 2026 · Dan*

> Copy of the "Governance Intake Platform: Design Document (Draft v0.1)" Claude Doc ([source](https://claude.ai/code/artifact/9e1c0796-d0b3-4597-96c2-cd668dcc27fd)), pasted as text on 2026-11-27 and reformatted to Markdown. Diagrams from the original are not included. Treat the source doc as authoritative; do not edit this copy by hand except to re-sync it.

## Summary

We will build one governance front door: an AI intake agent that collects facts about a project once, a deterministic rules engine that decides which governance domains apply, and a workflow layer that routes the resulting requirements and approvals from conception through launch and into run state.

Subject-matter experts (SMEs) in each domain own their own content through no-code tools: (a) applicability rules that say when their domain is triggered, and (b) the requirements, policies and approval chains that apply once it is. This draft proposes starter rule packs for twelve domains: Privacy, Information Security, Legal, Procurement and Vendor Risk, AI Governance, Finance, Records, Regulatory and Compliance (including accessibility), HR, Enterprise Risk and Operational Resilience, Data Governance, and Enterprise Architecture.

**Status**: Draft v0.1 for discussion. The rule packs are illustrative defaults, not legal advice. Each domain owner must validate them against company policy and the laws of the Canadian provinces where the company operates (scope: the ten provinces and the federal laws that apply in them).

**Core design principle**: the AI agent gathers and explains; it never decides. Applicability is computed by versioned, explainable rules. Approvals are granted only by named humans.

## Problem, goals and non-goals

Today, project teams discover governance requirements late, one function at a time. Each function runs its own intake form, asks overlapping questions, and learns about projects when it is expensive to change them. Nobody holds a single view of which reviews a project needed, which were done, and which obligations continue after launch.

**Baseline metrics to collect during discovery** (open question: current values):
- Median days from project kickoff to all approvals complete
- Share of projects where a required review started after build began
- Incidents or audit findings caused by a missed review
- SME hours spent per project on triage and chasing information

### Goals

1. One intake per project. Every fact is asked once and reused by every domain.
2. Correct triggering. Every applicable domain is identified at intake, with an explanation traceable to a rule.
3. SME ownership. Domain owners change their rules and requirements without engineering help.
4. Lifecycle coverage. Obligations continue after launch, and material changes re-trigger review.
5. Audit-ready evidence. For any project, show what applied, which rule version said so, who approved, and when.

### Non-goals (v1)

- The agent does not give legal advice, interpret policy beyond its source text, or grant approvals.
- Not a replacement for specialist tools (contract lifecycle management, vulnerability scanning, HRIS). The platform orchestrates and links to them.
- Not a policy-authoring or document-management system. Policies live in the policy repository; the platform references them by version.

## Users and personas

| Persona | Who | Primary need | Key screens |
|---|---|---|---|
| Requester | Product manager, project lead, business sponsor | Know early what is required, get to launch without surprises | Intake chat, project dashboard, task list |
| Domain SME / policy owner | Privacy counsel, security architect, procurement lead, etc. | Be triggered on the right projects with the right facts; control own rules | Rule builder, review queue, back-test, domain reports |
| Approver | Manager, CPO, CISO, GC, CFO delegate | Decide quickly with a complete record | Approval inbox, decision record |
| Governance admin | Central governance / PMO / Legal Ops | Own the attribute taxonomy, onboard domains, monitor health | Admin dashboard, taxonomy manager, domain registry, global reports |
| Auditor / executive | Internal audit, risk committee, board | Prove control operation; see portfolio risk | Evidence packs, global reports, read-only dashboard summary |
| Delivery team member | Engineer, designer, analyst | Know which controls to build in and what evidence to attach | Requirements checklist, evidence upload |

## Core concepts and domain model

The engine reads only confirmed attributes and the domains' published rules; everything a project owes flows out of that evaluation and is attached to the project.

| Concept | Definition |
|---|---|
| Project | Any initiative going through intake, from idea to retirement |
| Attribute | A typed fact about a project from the shared taxonomy (e.g., data.personal_info) |
| Domain | A governance area with an owner, such as Privacy or Finance |
| Applicability rule | Versioned condition over attributes that triggers a domain at a tier |
| Requirement | A task the project must complete: assessment, control, document, training or attestation |
| Approval chain | Ordered or parallel human sign-offs, resolved by role |
| Obligation | A recurring requirement that continues after launch |
| Evidence | Artefacts proving a requirement was met (files, links, system records) |
| Exception | A dated, approved waiver of a requirement, with a named risk owner |

## Shared project attribute taxonomy

Every domain writes rules against one shared vocabulary of project facts. The governance admin owns it; SMEs propose additions through change control. Each attribute has an ID, type, allowed values, a plain-language question the agent can ask, and a list of rules that consume it. Domain-specific attributes (in each domain pack below) extend this core set.

| Group | Attribute ID | Type | Allowed values / notes |
|---|---|---|---|
| Basics | proj.type | enum | new product, feature change, internal tool, process change, vendor onboarding, partnership, M&A, research/pilot |
| Basics | proj.stage | enum | idea, discovery, design, build, pre-launch, live, retiring |
| Basics | proj.launch_date | date | target launch |
| Basics | proj.audience | multi-enum | internal employees, contractors, B2B customers, consumers, minors, public sector clients |
| Basics | proj.customer_facing | boolean |   |
| Geography | geo.user_jurisdictions | multi-enum | AB, BC, MB, NB, NL, NS, ON, PE, QC, SK (see Jurisdiction coverage) |
| Geography | geo.data_locations | multi-enum | where data is stored or processed |
| Geography | geo.cross_border_transfer | boolean | derived: data leaves its province of collection or leaves Canada |
| Data | data.personal_info | boolean | any information about an identifiable individual |
| Data | data.categories | multi-enum | contact, identifiers, financial, health, biometric, precise location, children's, employee, behavioural/usage, inferred/profiles |
| Data | data.subject_volume | number band | <1k, 1k-100k, 100k-1M, >1M individuals |
| Data | data.classification | enum | public, internal, confidential, restricted |
| Data | data.new_use_of_existing | boolean | existing data used for a new purpose |
| Technology | tech.hosting | enum | on-prem, company cloud, vendor SaaS, hybrid |
| Technology | tech.internet_exposed | boolean |   |
| Technology | tech.auth_change | boolean | new or changed authentication / access model |
| Technology | tech.payment_flow | boolean | handles card or payment data |
| AI | ai.uses_ai | boolean | includes ML, generative AI or automated decisioning |
| AI | ai.role | enum | internal productivity, customer-facing assistant, decision support, automated decision |
| AI | ai.decision_domain | multi-enum | employment, credit/financial, insurance, health, housing, education, access to services, none |
| AI | ai.model_source | enum | in-house, third-party API, open-weight, embedded in vendor product |
| Third parties | vendor.involved | boolean |   |
| Third parties | vendor.new | boolean | not on approved-vendor list |
| Third parties | vendor.data_access | enum | none, internal, confidential, restricted |
| Commercial | fin.total_cost | money band | <$25k, $25k-$250k, $250k-$1M, >$1M (CAD) |
| Commercial | fin.revenue_impact | boolean | new pricing, revenue or billing model |
| Commercial | fin.capex | boolean |   |
| Commercial | legal.new_contract | boolean | new or amended third-party agreement |
| Commercial | legal.customer_terms_change | boolean |   |
| Content & UX | ux.new_digital_interface | boolean | new web, mobile, kiosk or document output |
| Content & UX | ux.marketing_claims | boolean | public claims about product or performance |
| People | hr.affects_employees | boolean | changes to employee monitoring, roles, workforce, or employee data |
| People | hr.headcount_change | boolean |   |
| Records | rec.creates_records | boolean | creates business records (transactions, decisions, communications) |
| Records | rec.new_repository | boolean | new system of record |

Rules for the taxonomy: attributes are additive and never silently redefined; a meaning change creates a new attribute and deprecates the old one. Derived attributes (e.g., geo.cross_border_transfer) are computed, never asked. Each attribute records its source: requester answer, agent extraction (confirmed), or system of record.

## Jurisdiction coverage: Canadian provinces

The platform covers the ten provinces only. Territories and foreign jurisdictions (US, UK, EU and others) are out of scope for v1; a project with users or data outside the provinces is flagged for manual Legal review rather than evaluated by rules.

Federal law applies in every province and is the baseline for all domain packs: PIPEDA (commercial activity in provinces without a substantially similar law, interprovincial and cross-border transfers, and federally regulated businesses), CASL, the Competition Act, the Income Tax and Excise Tax Acts, and, for federally regulated employers, the Canada Labour Code, federal Pay Equity Act and Accessible Canada Act. Watch item: federal Bill C-36 (Protecting Privacy and Consumer Data Act) was introduced June 15, 2026 and is not yet law.

Jurisdiction attributes (part of the shared taxonomy):
- geo.user_jurisdictions (multi-enum): AB, BC, MB, NB, NL, NS, ON, PE, QC, SK
- geo.employee_provinces (multi-enum): provinces where affected employees work
- geo.data_locations (multi-enum): province(s) where data is stored or processed, or outside Canada
- geo.cross_border_transfer (derived): data leaves its province of collection or leaves Canada
- geo.federally_regulated (boolean): banking, telecom, interprovincial transport or other federal undertaking
- geo.out_of_scope (derived): any user, employee or entity outside the ten provinces; routes to manual Legal review

### Provincial law matrix

**Status** as reported in the sources below as of November 2026. Domain owners must verify each cell before rules go live.

| Province | Private-sector privacy | Health information | Accessibility (private-sector reach) | Pay equity / pay transparency | Other provincial flags |
|---|---|---|---|---|---|
| Alberta | Alberta PIPA (applies instead of PIPEDA in-province); government consultation on modernizing PIPA in 2026 | Health Information Act (amendments in force June 2026) | No accessibility statute | No private-sector requirement | Breach notification to Alberta OIPC; employee personal information covered by PIPA |
| British Columbia | BC PIPA (applies instead of PIPEDA in-province) | Covered largely by PIPA for private-sector health providers | Accessible British Columbia Act (public sector focus) | Pay Transparency Act: pay ranges in job postings; annual reports for employers with 50+ employees from Nov 1, 2026 | PST registration; employee personal information covered by PIPA |
| Manitoba | PIPEDA | Personal Health Information Act | Accessibility for Manitobans Act: customer service, employment and information and communication standards apply to private organizations | No private-sector requirement | PST registration |
| New Brunswick | PIPEDA | Personal Health Information Privacy and Access Act | Accessibility Act (2024); standards still in development | Pay ranges in job postings reported in force; reporting provisions not yet in force | Official Languages Act mainly binds public bodies |
| Newfoundland and Labrador | PIPEDA | Personal Health Information Act | Accessibility Act (2021); no private-sector standards yet | Pay Equity and Pay Transparency Act enacted; check which parts are in force |   |
| Nova Scotia | PIPEDA | Personal Health Information Act | Accessibility Act (2017); standards rolling out by sector | No pay-range posting rule; wage-history questions banned | Public-sector clients may impose data-residency terms |
| Ontario | PIPEDA | Personal Health Information Protection Act (PHIPA) | AODA and its Integrated Accessibility Standards: customer service, information and communications (WCAG), employment | Pay Equity Act (employers with 10+ employees); pay ranges and AI-use disclosure in job postings for employers with 25+ (since Jan 1, 2026) | ESA electronic monitoring policy (25+ employees); Consumer Protection Act, 2002 in force, Consumer Protection Act, 2023 not yet proclaimed |
| Prince Edward Island | PIPEDA | Health Information Act | No private-sector standards identified (verify) | Pay ranges in job postings |   |
| Quebec | Private-sector privacy act as amended by Law 25 (applies instead of PIPEDA in-province): PIAs, transfer assessments, privacy by default, automated-decision notice, parental consent under 14 | Act respecting health and social services information | Act to secure handicapped persons' rights (limited private-sector duties) | Pay Equity Act (employers with 10+ employees) | Charter of the French Language (as amended by Bill 96); biometric database disclosure to the CAI; QST registration; Consumer Protection Act |
| Saskatchewan | PIPEDA | Health Information Protection Act | Accessible Saskatchewan Act (government only for now) | No private-sector requirement | PST registration |

Every province also has its own consumer protection, employment standards, human rights, occupational health and safety and labour relations statutes. Each domain owner maps those per province in their pack.

## Rules engine and rule language

Applicability is computed by a deterministic engine over confirmed project attributes, never by the LLM. Same facts plus same rule versions always produce the same result, and every result can be explained line by line.

### Rule schema

```text
rule_id:        PRV-02
domain:         Privacy
version:        4  (effective 2026-09-01; supersedes v3)
owner:          Privacy Office
status:         draft | in_review | active | retired
when:           ALL/ANY of conditions over attributes
                (=, !=, in, contains_any, >=, is_unknown)
unknown_policy: trigger | ask_sme | ignore   (default: trigger)
then:
  requirements: [PIA-STD]                      # from the requirement library
  approvals:    [chain: PRV-CHAIN-HIGH]
  sets:         { prv.risk_tier: high }        # derived attributes other rules may read
  obligations:  [OBL-PIA-ANNUAL]
  stage_gate:   design                         # must be cleared before this stage exits
rationale:      plain-language reason shown to requesters
source:         Privacy Policy s.4.2 (v2026.1); PIPEDA; Quebec Law 25
tests:          [TC-PRV-02-a, TC-PRV-02-b]
```

### Evaluation

1. Evaluate all active rules against current attributes; record rule ID and version for every hit and every near-miss.
2. Apply derived attributes (sets), then re-evaluate until no new attributes are set (fixed point; cycle detection blocks publishing any rule that could loop).
3. Merge outputs. Requirements are de-duplicated by ID; approvals with the same approver at the same stage are merged; the stricter of two conflicting settings wins (e.g., the shorter retention review interval).
4. Unknown attributes follow each rule's unknown_policy. Default is trigger, so missing information produces over-review, never a silent miss.

### Versioning and change

- Rules are immutable once active; edits create a new version with an effective date.
- Projects keep a frozen snapshot of the rule versions applied at each gate.
- When a rule changes, impact analysis lists in-flight and live projects that newly trigger or no longer trigger. The owner chooses: apply to new projects only, apply to in-flight projects, or apply to all (creates remediation tasks).

### Conflict handling

Publishing checks for: contradictory requirements across domains, duplicate approvals, circular triggers, and attributes that no rule reads (dead questions). Conflicts block publishing until the owning SMEs resolve them or the governance admin records a precedence decision.

## Intake agent design

The agent's job is to fill the attribute set accurately with minimum effort from the requester, then explain the result. It proposes facts; the requester confirms them; the engine decides.

### Conversation flow

1. Open. Requester describes the project in free text and may attach a brief, PRD, slide deck or vendor quote.
2. Extract. Agent pre-fills attributes from the text and documents, each with a confidence score and the quote it relied on.
3. Ask only what matters. Agent asks for missing attributes in order of how many rules they could change (information gain), skipping questions no active rule depends on. Questions use plain language and examples ("Will you collect anything that could identify a person, such as an email address or device ID?").
4. Confirm. Requester reviews a one-page fact sheet and attests it is accurate. Low-confidence extractions are highlighted.
5. Result. Engine output is shown as: triggered domains, requirements by stage, approvals and owners, expected timeline, and "why" links to each rule's rationale.
6. Continue. The agent stays available on the project: answers policy questions, nudges on overdue tasks, and asks whether facts have changed at each stage gate.

### Policy Q&A

Answers come only from the approved policy library (retrieval with citations to policy section and version). If the library does not answer the question, the agent says so and routes it to the domain SME. The agent labels its answers as guidance, not legal advice or an approval.

### Guardrails

- Cannot change rule outcomes, waive requirements, or record approvals.
- Cannot mark an attribute confirmed without the requester's action.
- Ambiguity defaults to the conservative value (or unknown, which triggers review).
- Detects inconsistency: if the brief mentions "customer emails" but the requester answers no to personal information, the agent flags the conflict.
- Cross-checks answers against systems of record where possible (vendor list, data catalog, CMDB).
- All agent turns, extractions and confirmations are logged for audit.

### Evaluation

Before launch and on every model or prompt change, run a test set of at least 100 historical and synthetic project descriptions with known correct attributes. Track: attribute extraction precision and recall, false-negative rate on triggering attributes (target: 0 on the gold set for personal info, AI use and new vendor), questions asked per intake, and citation accuracy in Q&A.

## SME authoring experience

SMEs maintain their domain without engineering help, but every change is tested and reviewed before it reaches live projects.

Domain registry. Each domain has an owner, a backup owner, a description, a policy library, a requirement library, approval chains, and its rules.

Rule builder. A form-based condition editor (pick attribute, pick operator, pick values) with AND/OR groups, shown alongside a plain-English rendering ("This rule applies when the project collects health or biometric data from more than 1,000 people"). Power users can edit the underlying rule text directly.

Requirement library. Reusable items: assessment (PIA, threat model, AIA), control (encryption at rest), document (DPA signed), training, or attestation. Each has instructions, a template, evidence required, an owner, and a target stage.

Approval chain builder. Sequential and parallel steps; approvers resolved by role from the org chart (e.g., "CISO delegate for the requester's business unit"); conditional steps ("add CFO if fin.total_cost > $1M"); SLA per step; escalation path; delegation rules.

Testing. Each rule needs at least two test cases (one that triggers, one that does not) before it can be submitted.

Back-testing. "Run against history" shows which past projects the draft rule would have triggered, compared with the current version. This is the main defence against over- and under-triggering.

Change control. Draft, then peer review by a second SME in the same domain, then publish. Material changes (new approval chains, new triggers affecting more than 10% of projects) also need governance admin sign-off.

Health signals. Alerts for rules not reviewed in 12 months, requirements frequently waived, approval steps that routinely breach SLA, and questions that requesters often answer "unknown".

Rule maintenance assistant. An AI assistant helps owners draft, test and update rules; every change still goes through the change control above. See Additional capabilities.

## Workflow, approvals and lifecycle

Each requirement is tied to the stage it must clear; a project cannot pass a gate while a gating requirement or approval is open, unless an approved exception covers it.
- Parallel by default. Domains review at the same time; sequencing applies only inside an approval chain or where a rule says one output feeds another (e.g., PIA before CPO approval).
- Gates. Design gate clears assessments that shape the build (PIA, threat model, AIA, vendor selection). Pre-launch gate clears tests, contracts, notices and final sign-offs.
- SLAs and escalation. Each approval step has an SLA (proposed: 5 business days Tier 2, 10 Tier 3); breaches notify the approver, then their manager, then the domain owner.
- Delegation. Approvers can delegate by role and date range; delegations are logged.
- Exceptions. Requester proposes, domain owner approves, business owner accepts residual risk; every exception has an expiry and creates a review task.
- Re-triggering. A change to any attribute, or a newly published rule version applied to live projects, re-runs evaluation. New requirements are added; satisfied ones stay satisfied unless the rule says otherwise.
- Run state. Obligations generate tasks on schedule, assigned to the system owner, with evidence required to close.

## Domain rule packs: how to read them

Each of the twelve packs below is a proposed starter set for the domain owner to accept, edit or reject. Every pack has the same five parts:
1. Domain taxonomy. Attributes the domain adds to the shared set, usually asked only once the domain is triggered.
2. Applicability rules. When the domain is triggered and at what tier. Rule IDs are referenced in the cross-domain map.
3. Policies and standards. The policy documents the requirements point to (to be mapped to the company's actual policy library).
4. Approval chain. Who approves, by tier.
5. Run-state obligations. What continues after launch.

Most domains use three tiers. Tier 1 (notify): self-service checklist and attestation; no SME review. Tier 2 (review): SME review and sign-off. Tier 3 (escalate): senior approver and formal assessment.

Legal references cover the ten Canadian provinces and the federal laws that apply in them; see Jurisdiction coverage for the province-by-province map. Thresholds (dollar amounts, volumes) are placeholders to be set by each owner. Nothing here is legal advice; counsel must validate each pack.

### 1. Privacy

Owner: Chief Privacy Officer. Triggered whenever personal information is collected, used, disclosed or retained in a new way.

#### Domain taxonomy

- prv.purposes (multi-enum): service delivery, analytics, marketing, personalization, profiling, research, fraud/security, sale or sharing for value
- prv.consent_model (enum): express consent, implied consent, opt-out, consent exception (e.g., required by law, business transaction)
- prv.sharing (multi-enum): none, service providers, affiliates, partners, sale or sharing for advertising
- prv.automated_decision (boolean): decision about an individual made by technology without human review
- prv.tracking_tech (boolean): cookies, pixels, SDKs, device fingerprinting
- prv.retention_defined (boolean)
- prv.dsr_impact (boolean): affects ability to fulfil access, correction or deletion requests
- prv.risk_tier (derived): low, medium, high

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| PRV-01 | data.personal_info = true | Privacy in scope. Tier 1: privacy checklist (notice, purpose, minimization, retention); governing law set per province (PIPEDA, Alberta PIPA, BC PIPA or Quebec act) |
| PRV-02 | data.categories contains any of health, biometric, financial, precise location, children's; OR data.subject_volume >= 100k; OR prv.automated_decision = true | Tier 3: full Privacy Impact Assessment; sets prv.risk_tier = high; if health data, check the provincial health information act |
| PRV-03 | geo.user_jurisdictions contains QC AND data.personal_info = true | PIA required (Quebec Law 25 requires a privacy impact assessment for certain projects); French-language notice review |
| PRV-04 | geo.cross_border_transfer = true | Transfer assessment; for QC data, a transfer PIA before communication outside Quebec; for data leaving Canada, notice of foreign processing and contractual safeguards |
| PRV-05 | data.new_use_of_existing = true | Purpose-compatibility review; consent refresh if purpose is not consistent with original notice |
| PRV-06 | prv.tracking_tech = true OR prv.purposes contains marketing | Cookie/consent banner review; CASL review for commercial electronic messages; for QC users, tracking and profiling functions off by default (Law 25) |
| PRV-07 | proj.audience contains minors OR data.categories contains children's | Tier 3; age-appropriate design review; parental consent where required (Quebec: under 14) |
| PRV-08 | prv.sharing contains sale or sharing for advertising | Tier 3; CPO approval; express consent expected for sensitive data |
| PRV-09 | vendor.involved = true AND vendor.data_access != none AND data.personal_info = true | Data processing agreement required (triggers PRC) |

#### Policies and standards

Privacy Policy (internal), Privacy Notice (external), Privacy by Design Standard, PIA Procedure, Data Retention Schedule, Cross-Border Transfer Procedure, Data Subject Request Procedure, Breach Response Plan. Laws mapped: PIPEDA (baseline, interprovincial and cross-border transfers); Alberta PIPA, BC PIPA and Quebec's private-sector act as amended by Law 25 (instead of PIPEDA within those provinces); provincial health information acts (see Jurisdiction coverage); CASL. Watch: federal Bill C-36 and the Alberta PIPA review.

#### Approval chain

- Tier 1: requester attestation, auto-logged; privacy analyst spot-checks 10%
- Tier 2: privacy analyst or counsel
- Tier 3: privacy counsel, then CPO

#### Run-state obligations

- Annual PIA refresh for Tier 3; on material change for all tiers
- Retention deletion jobs verified annually
- Keep the record of processing activities (RoPA) current
- DSR fulfilment tested before launch and annually
- Notice re-review when purposes or sharing change

### 2. Information Security

Owner: CISO. Triggered by any new system, data flow, integration, access model or vendor that touches non-public data.

#### Domain taxonomy

- sec.system_criticality (enum): low, medium, high, critical (business impact if unavailable)
- sec.integrations (multi-enum): none, internal APIs, external APIs, SSO/IdP, file transfer, webhooks
- sec.privileged_access (boolean): admin or production access for new users or service accounts
- sec.new_code (boolean): new code deployed to company environments
- sec.open_source_components (boolean)
- sec.encryption (multi-enum): in transit, at rest, customer-managed keys, none
- sec.logging_monitoring (boolean): security logs sent to SIEM
- sec.risk_tier (derived): low, medium, high

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| SEC-01 | data.classification in confidential, restricted OR tech.internet_exposed = true OR sec.new_code = true | Security in scope. Tier 1: secure-by-design checklist |
| SEC-02 | tech.internet_exposed = true AND data.classification in confidential, restricted | Tier 2: threat model and architecture review; penetration test before launch |
| SEC-03 | data.classification = restricted OR sec.system_criticality in high, critical OR tech.payment_flow = true | Tier 3: full security assessment; CISO approval |
| SEC-04 | tech.payment_flow = true | PCI DSS scoping and attestation |
| SEC-05 | tech.auth_change = true OR sec.privileged_access = true | IAM review: SSO, MFA, least privilege, access recertification set up |
| SEC-06 | vendor.involved = true AND vendor.data_access != none | Third-party security assessment (feeds PRC-03) |
| SEC-07 | sec.new_code = true | SAST/DAST and dependency scanning in pipeline; SBOM if sec.open_source_components |
| SEC-08 | ai.uses_ai = true AND ai.role in customer-facing assistant, automated decision | AI security review: prompt injection, data leakage, model access controls (paired with AIG) |

#### Policies and standards

Information Security Policy, Data Classification Standard, Secure Development Standard, Access Control Standard, Encryption Standard, Logging and Monitoring Standard, Vulnerability Management Standard, Third-Party Security Standard, Incident Response Plan. Frameworks: ISO/IEC 27001, NIST CSF, SOC 2 controls, PCI DSS.

#### Approval chain

- Tier 1: engineering lead attestation
- Tier 2: security architect
- Tier 3: security architect, then CISO (or delegate); residual risk acceptance by business owner

#### Run-state obligations

- Quarterly access recertification for privileged access
- Annual penetration test for Tier 3 internet-facing systems
- Vulnerability SLAs (e.g., critical patched within 14 days)
- Logs retained and monitored; incident runbook kept current
- Risk acceptances expire after 12 months

### 3. Legal (commercial, IP, regulatory, consumer)

Owner: General Counsel, with sub-owners for Commercial, IP and Litigation/Disputes. Triggered by new contracts, customer-facing changes, IP creation or use, and public claims. Regulatory obligations, licensing and French-language requirements sit with Regulatory and Compliance.

#### Domain taxonomy

- lgl.contract_type (multi-enum): customer terms, vendor/SaaS, partnership/JV, licensing in, licensing out, NDA, reseller/distribution, government contract
- lgl.non_standard_terms (boolean): departs from approved templates (liability, indemnity, IP ownership, exclusivity)
- lgl.ip_created (boolean): new patentable invention, brand, or significant copyrighted work
- lgl.third_party_content (boolean): uses third-party data, content, trademarks or open-source code in a product
- lgl.new_market_entry (boolean): new province or regulated industry
- lgl.consumer_offer (boolean): subscriptions, auto-renewals, promotions, contests
- lgl.competitor_interaction (boolean): information sharing or collaboration with a competitor

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| LGL-01 | legal.new_contract = true | Contract review; template use if available; Tier 2 if lgl.non_standard_terms |
| LGL-02 | legal.customer_terms_change = true OR lgl.consumer_offer = true | Customer terms and consumer protection review (each province's consumer protection act, e.g., Ontario's Consumer Protection Act, 2002 and the 2023 Act once proclaimed, Quebec's Consumer Protection Act; contest rules under the Competition Act) |
| LGL-03 | ux.marketing_claims = true | Advertising claims substantiation review (Competition Act misleading advertising, including environmental claims) |
| LGL-04 | lgl.ip_created = true | IP capture: invention disclosure, trademark clearance and filing decision |
| LGL-05 | lgl.third_party_content = true OR sec.open_source_components = true | Licence compliance review; open-source policy check (copyleft in distributed products is Tier 2) |
| LGL-07 | lgl.competitor_interaction = true | Tier 3: competition law review before any information exchange |
| LGL-09 | lgl.contract_type contains government contract | Public-sector terms, procurement integrity, security clearance needs |

#### Policies and standards

Contract Approval and Signing Authority Policy, Contract Templates and Playbooks, Open Source Policy, IP Policy and Invention Disclosure Procedure, Advertising and Claims Standard, Competition Law Compliance Policy.

#### Approval chain

- Tier 1: legal self-service (approved template, no edits) with Legal Ops log
- Tier 2: assigned commercial or IP counsel
- Tier 3: senior counsel or GC; signing per signing-authority matrix

#### Run-state obligations

- Contract obligations and renewal dates tracked (notice periods, SLAs, audit rights)
- Trademark and patent maintenance deadlines
- Legal input to regulatory filings owned by Regulatory and Compliance
- Claims re-substantiated when product changes

### 4. Procurement and Vendor Risk

Owner: Head of Procurement, with Third-Party Risk Management (TPRM). Triggered whenever the company buys from, licenses from, or shares data or access with a third party.

#### Domain taxonomy

- prc.spend_type (enum): one-time, subscription, usage-based, services/SOW
- prc.sole_source (boolean): no competitive process
- prc.criticality (enum): low, medium, high (impact if the vendor fails or exits)
- prc.subcontractors (boolean): vendor uses fourth parties to deliver
- prc.offshore_delivery (boolean): vendor personnel or processing outside Canada
- prc.concentration (boolean): vendor already supports other critical services
- prc.vendor_risk_tier (derived): low, medium, high

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| PRC-01 | vendor.involved = true | Procurement in scope; vendor onboarding (legal entity, banking, tax, sanctions screening) if vendor.new |
| PRC-02 | fin.total_cost >= $25k | Competitive process (3 quotes) or sole-source justification; >= $250k formal RFP |
| PRC-03 | vendor.data_access in confidential, restricted OR prc.criticality = high | TPRM Tier 3: security questionnaire, SOC 2 / ISO 27001 evidence, financial viability, BCP/DR review |
| PRC-04 | vendor.data_access = internal | TPRM Tier 2: lite questionnaire |
| PRC-05 | prc.criticality = high | Exit plan and alternative supplier identified; contract must include termination assistance |
| PRC-06 | prc.offshore_delivery = true OR prc.subcontractors = true | Fourth-party and location review; triggers PRV-04 if personal info |
| PRC-07 | vendor.involved = true AND ai.model_source in third-party API, embedded in vendor product | AI vendor terms review: training on company data, retention, output IP (with AIG and LGL) |

#### Policies and standards

Procurement Policy, Signing Authority Matrix, Third-Party Risk Management Standard, Supplier Code of Conduct, Sanctions Screening Procedure, Vendor Exit Planning Standard. Where relevant, sector guidance such as OSFI Guideline B-10 (third-party risk) for federally regulated financial institutions.

#### Approval chain

- Tier 1: budget owner + procurement buyer
- Tier 2: procurement manager + TPRM analyst
- Tier 3: Head of Procurement + TPRM lead + business executive accepting residual risk

#### Run-state obligations

- Vendor reassessment: annually for high risk, every 2 years for medium, every 3 for low
- Contract renewal review 90 days before auto-renewal
- Performance and SLA review quarterly for critical vendors
- Continuous monitoring (security ratings, adverse news, sanctions)
- Offboarding: data return or destruction certificate, access revocation

### 5. AI Governance

Owner: AI Governance Lead (often reporting to the CPO, GC or Chief Data Officer), supported by a cross-functional AI review board. Triggered by any build, purchase or use of AI that affects customers, employees or decisions.

#### Domain taxonomy

- aig.use_case_category (enum): productivity/internal, content generation, customer interaction, recommendation, classification/scoring, autonomous agent, biometric identification
- aig.human_oversight (enum): human decides, human reviews before effect, human can override after, none
- aig.impact_on_individuals (enum): none, low, significant (legal or similarly significant effect)
- aig.training_data_source (multi-enum): company data, customer data, licensed, public web, synthetic, vendor pre-trained only
- aig.generates_content_for_public (boolean)
- aig.autonomy (boolean): agent can take actions in systems without step-by-step approval
- aig.impact_level (derived): prohibited by policy, high, medium, low
- aig.risk_tier (derived): low, medium, high, prohibited

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| AIG-01 | ai.uses_ai = true | Register in AI inventory; Tier 1 acceptable-use checklist |
| AIG-02 | ai.role in customer-facing assistant OR aig.generates_content_for_public = true | Tier 2: transparency notice (users told they interact with AI); output safety testing; labelling of synthetic content |
| AIG-03 | ai.decision_domain != none OR aig.impact_on_individuals = significant | Tier 3: Algorithmic Impact Assessment, bias and fairness testing, explainability, human review path, appeal mechanism; triggers PRV-02 |
| AIG-04 | geo.user_jurisdictions contains QC AND prv.automated_decision = true | Tier 3: Quebec Law 25 automated-decision duties (inform the person, explain the factors on request, allow review by a human) |
| AIG-05 | aig.use_case_category = biometric identification | Tier 3; presumed high risk; legal review for prohibited practices; Quebec biometric database disclosure to the CAI where applicable |
| AIG-06 | aig.autonomy = true | Tier 2 minimum: action scope limits, kill switch, logging, red-team testing |
| AIG-07 | aig.training_data_source contains customer data OR company data | Data rights review (contract and notice permit training?); triggers PRV-05 |
| AIG-08 | hr.affects_employees = true AND ai.uses_ai = true | Employee-facing AI review (with HR); Ontario ESA job-posting AI disclosure for employers with 25+ employees (in force since Jan 1, 2026) |

#### Policies and standards

AI Policy and Acceptable Use Standard, AI Risk Classification Standard, Algorithmic Impact Assessment template, Model Documentation (model card) Standard, AI Testing and Red-Teaming Standard, AI Incident Procedure. Laws and frameworks: Quebec Law 25 automated-decision rules, Ontario ESA AI hiring disclosure, provincial privacy and human rights law; ISO/IEC 42001 as the voluntary management-system standard; monitor federal AI and privacy bills.

#### Approval chain

- Tier 1: AI inventory registration, auto-approved
- Tier 2: AI governance lead
- Tier 3: AI review board (Legal, Privacy, Security, Data Science, business owner); executive sign-off for significant-impact uses
- Prohibited: blocked; escalation only to GC

#### Run-state obligations

- Performance, drift and bias monitoring at a set frequency (e.g., monthly for Tier 3)
- Re-assessment on model change, new training data or new use
- AI incident logging and review
- Annual AI inventory attestation by system owners

### 6. Finance

Owner: CFO, delegated to FP&A, Controllership, Tax and Treasury. Triggered by spend, revenue, accounting or tax impact.

#### Domain taxonomy

- fin.budgeted (boolean): spend is in the approved budget
- fin.cost_centre (text): resolved from org data
- fin.multi_year_commitment (boolean)
- fin.revenue_model (enum): new SKU, pricing change, new billing model, bundling, discounting program, none
- fin.new_legal_entity_or_province (boolean)
- fin.capitalizable (boolean): software development or asset that may be capitalized
- fin.lease (boolean): contract may contain a lease
- fin.financial_reporting_system (boolean): touches a system that feeds the general ledger or financial statements

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| FIN-01 | fin.total_cost > $0 | Budget check against cost centre; Tier 1 if budgeted and under $25k |
| FIN-02 | fin.budgeted = false OR fin.total_cost >= $250k | Tier 2: business case and FP&A approval |
| FIN-03 | fin.total_cost >= $1M OR fin.multi_year_commitment = true | Tier 3: CFO approval; investment committee if above board-delegated limits |
| FIN-04 | fin.revenue_impact = true OR fin.revenue_model != none | Revenue recognition review (IFRS 15, or ASPE for private enterprises), billing and pricing approval |
| FIN-05 | fin.capitalizable = true OR fin.capex = true OR fin.lease = true | Accounting treatment review (capitalization, IFRS 16 or ASPE leases) |
| FIN-06 | fin.new_legal_entity_or_province = true OR geo.user_jurisdictions adds a province | Tax review: GST/HST, QST (Quebec) and PST (BC, MB, SK) registration; provincial income allocation; payroll and withholding by province |
| FIN-07 | fin.financial_reporting_system = true | ICFR control impact assessment (NI 52-109 certification for reporting issuers); IT general controls review |

#### Policies and standards

Delegation of Financial Authority, Budget and Forecasting Policy, Capital Expenditure Policy, Revenue Recognition Policy, Pricing and Discount Approval Policy, Tax Policy, Expense and Purchasing Card Policy, Internal Controls over Financial Reporting framework.

#### Approval chain

- Tier 1: budget owner
- Tier 2: FP&A business partner + budget owner's VP
- Tier 3: CFO (and CEO/board per delegation matrix)
- Revenue, tax and accounting reviews: Controller, Tax lead as relevant

#### Run-state obligations

- Actuals vs business case review at 6 and 12 months
- Multi-year commitment tracked in forecast
- ICFR controls tested per annual cycle
- Tax filings and registrations maintained

### 7. Records and Information Management

Owner: Records Manager (often within Legal or Compliance). Triggered when a project creates business records or a new place to keep them.

#### Domain taxonomy

- rec.record_classes (multi-enum): financial/transactional, contracts, customer communications, HR/personnel, product/technical, regulatory submissions, board/corporate, marketing
- rec.retention_rule_mapped (boolean): each record class mapped to the retention schedule
- rec.legal_hold_capable (boolean): system can suspend deletion for litigation hold
- rec.ediscovery_exportable (boolean): records can be searched and exported
- rec.format (multi-enum): structured data, documents, email/chat, audio/video, logs
- rec.system_of_record (boolean): this system is the authoritative copy

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| REC-01 | rec.creates_records = true | Map record classes to the retention schedule; Tier 1 |
| REC-02 | rec.new_repository = true OR rec.system_of_record = true | Tier 2: records review: retention configuration, legal hold capability, e-discovery export, defensible deletion |
| REC-03 | rec.format contains email/chat OR audio/video | Communications retention review (incl. recording-consent rules) |
| REC-04 | proj.stage = retiring | Decommissioning plan: migrate or destroy records per schedule; legal hold check; destruction certificate |
| REC-05 | rec.record_classes contains regulatory submissions OR financial/transactional | Regulatory minimum retention check (e.g., tax records retained at least 6 years under the Income Tax Act) |
| REC-06 | data.personal_info = true AND rec.creates_records = true | Align retention with privacy minimization (paired with PRV-01) |

#### Policies and standards

Records Management Policy, Records Retention Schedule, Legal Hold Procedure, E-Discovery Readiness Standard, Information Disposal Standard, Email and Messaging Retention Standard.

#### Approval chain

- Tier 1: system owner attestation
- Tier 2: Records Manager
- Destruction of records under potential hold: Litigation counsel

#### Run-state obligations

- Annual retention attestation by system owner
- Deletion jobs verified; exception log reviewed
- Legal hold tests at least annually
- Schedule updates propagated to configured systems

### 8. Regulatory and Compliance

Owner: Chief Compliance Officer, with sub-leads for Regulatory Affairs and Licensing, Accessibility, French Language, and Financial Crime (anti-bribery and sanctions). Triggered when a project touches a regulated activity, needs a licence, filing or regulator notice, affects regulatory reporting, involves sanctions or public officials, serves Quebec, or creates or changes a digital interface or customer service channel. Legal gives legal interpretation; Regulatory and Compliance owns the obligations, filings, regulator relationships and compliance testing.

#### Domain taxonomy

- rc.regulated_activity (multi-enum): financial services, insurance, health, telecom, consumer lending, gambling, energy, transportation, product or food safety, none
- rc.licence_or_registration (boolean): needs a new or amended licence, registration or regulatory approval
- rc.regulator_notification (boolean): requires notice to, or approval from, a regulator before launch
- rc.regulatory_reporting (boolean): changes data or processes that feed regulatory reports or filings
- rc.sanctions_export (boolean): export-controlled technology, or sanctioned countries, entities or people
- rc.public_official_interaction (boolean): dealings with government officials, or agents and intermediaries who deal with them
- acc.interface_types (multi-enum): website, web app, mobile app, kiosk/hardware, PDF/documents, video/audio, email templates, voice/IVR
- acc.audience_scope (enum): public, customers only, employees only
- acc.third_party_component (boolean): embeds a vendor widget or platform
- acc.service_change (boolean): changes how customers or employees receive service (in person, phone, online)
- acc.wcag_target (derived): WCAG 2.1 AA default; WCAG 2.2 AA recommended
- rc.risk_tier (derived): low, medium, high

#### Applicability rules

##### Regulatory and licensing

| Rule | Triggers when | Result |
|---|---|---|
| RC-01 | rc.regulated_activity != none OR lgl.new_market_entry = true | Tier 3: regulatory analysis by province and federally; licensing or registration requirements (moved from Legal) |
| RC-02 | rc.licence_or_registration = true OR rc.regulator_notification = true | Tier 3: filing plan and regulator engagement; pre-launch gate blocked until approval is received |
| RC-03 | rc.regulatory_reporting = true | Tier 2: reporting impact assessment; controls and testing (paired with DG-03 and FIN-07) |
| RC-04 | rc.sanctions_export = true OR vendor.new = true | Sanctions screening of new parties; Tier 3 export control classification if rc.sanctions_export (Export and Import Permits Act, Special Economic Measures Act) |
| RC-05 | rc.public_official_interaction = true | Tier 2: anti-bribery due diligence on intermediaries (Corruption of Foreign Public Officials Act; Criminal Code) |
| RC-06 | geo.user_jurisdictions contains QC OR geo.employee_provinces contains QC | French-language review: customer materials, product interfaces, contracts and workplace documents (Charter of the French Language, as amended by Bill 96) (moved from Legal) |

##### Accessibility

| Rule | Triggers when | Result |
|---|---|---|
| RC-07 | ux.new_digital_interface = true | Tier 1: accessibility design checklist; automated testing in the pipeline |
| RC-08 | ux.new_digital_interface = true AND acc.audience_scope = public | Tier 2: manual audit against WCAG AA before launch (AODA Integrated Accessibility Standards in Ontario; information and communication standard under the Accessibility for Manitobans Act; ACA for federally regulated entities; other provinces per Jurisdiction coverage) |
| RC-09 | acc.interface_types contains video/audio | Captions, transcripts and audio description plan |
| RC-10 | acc.third_party_component = true | Vendor accessibility conformance report (ACR/VPAT) required in procurement (feeds PRC-01) |
| RC-11 | acc.service_change = true OR (geo.user_jurisdictions contains ON or MB AND proj.customer_facing = true) | Accessible customer service check: staff training, alternate formats, feedback process (AODA, Manitoba AMA) |
| RC-12 | hr.affects_employees = true AND ux.new_digital_interface = true | Employee-facing tool accessibility; accommodation process confirmed |

#### Policies and standards

Compliance Management Framework, Regulatory Licensing and Filings Procedure, Regulatory Reporting Controls Standard, Anti-Bribery and Corruption Policy, Export Controls and Sanctions Policy (both moved from Legal), French Language Compliance Standard, Accessibility Policy, Digital Accessibility Standard (WCAG 2.1/2.2 AA), Accessible Procurement Standard, Accessible Customer Service Standard, Multi-Year Accessibility Plan. The regulatory change tracking feature (see Additional capabilities) feeds this domain's rule review queue.

#### Approval chain

- Tier 1: compliance self-assessment by the project lead (for accessibility, design and engineering lead attestation with a passing automated test)
- Tier 2: sub-area lead (regulatory affairs, accessibility, French language or financial crime)
- Tier 3: Chief Compliance Officer; General Counsel joins for regulator filings and licence applications
- Accessibility exceptions: remediation plan with a dated exception accepted by the business owner

#### Run-state obligations

- Licence renewals and regulatory filings tracked on a compliance calendar
- Compliance monitoring and testing plan for regulated activities; findings tracked to closure
- Regulatory reporting controls tested on the annual cycle
- Sanctions rescreening of counterparties on a set schedule
- Accessibility scans on every release; manual audit annually for public properties; AODA and Manitoba AMA compliance reporting on schedule; track NS, NB, NL and SK standards as they come into force
- Exceptions expire and are re-reviewed

### 9. HR and People

Owner: CHRO, delegated to HR Business Partners, Employee Relations, Total Rewards and HR Compliance. Triggered when a project affects employees, contractors, candidates or workforce structure.

#### Domain taxonomy

- hr.population (multi-enum): employees, contractors, candidates, interns, unionized employees
- hr.monitoring (boolean): monitors employee activity, location, communications or productivity
- hr.role_change (enum): none, job duties change, reorganization, role elimination, new roles
- hr.workforce_impact_size (number band): <10, 10-49, 50+ employees affected
- hr.contractor_use (boolean): engages individual contractors or staffing agencies
- hr.compensation_impact (boolean): changes to pay, incentives, benefits
- hr.hiring_tools (boolean): affects recruiting, screening or assessment
- hr.training_required (boolean)

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| HR-01 | hr.affects_employees = true | HR in scope; Tier 1 HRBP notification |
| HR-02 | hr.monitoring = true | Tier 2: monitoring review; update the Electronic Monitoring Policy disclosure (required for Ontario employers with 25+ employees); in AB, BC and QC, employee personal information rules under provincial privacy law also apply; triggers PRV-02 |
| HR-03 | hr.role_change in reorganization, role elimination OR hr.workforce_impact_size >= 50 | Tier 3: Employee Relations and employment counsel; group termination notice rules under each affected province's employment standards law (thresholds differ by province); communications plan |
| HR-04 | hr.population contains unionized employees | Labour relations review; collective agreement notice and consultation obligations |
| HR-05 | hr.contractor_use = true | Worker classification review (employee vs contractor); background check and onboarding requirements |
| HR-06 | hr.compensation_impact = true | Total Rewards review; pay equity (Ontario and Quebec Pay Equity Acts for employers with 10+ employees; federal Pay Equity Act if federally regulated) and pay transparency (pay ranges in job postings in BC, ON, PE and NB; BC pay reports for 50+ employees) |
| HR-07 | hr.hiring_tools = true | Recruiting compliance: human rights code review, job-posting disclosure requirements, AI screening review (triggers AIG-08) |
| HR-08 | hr.training_required = true | Learning plan; mandatory training assignment and completion tracking |

#### Policies and standards

Code of Conduct, Electronic Monitoring Policy, Workplace Human Rights and Anti-Harassment Policy, Accommodation Policy, Recruiting and Selection Policy, Contingent Worker Policy, Compensation Policy, Organizational Change Procedure, Health and Safety Policy. Laws mapped: each province's employment standards, human rights, occupational health and safety, labour relations and pay equity/transparency statutes (e.g., Ontario ESA, Human Rights Code, OHSA; Quebec Act respecting labour standards and Charter of the French Language for workplace language); Canada Labour Code for federally regulated employees.

#### Approval chain

- Tier 1: HRBP notified
- Tier 2: HRBP + HR Compliance
- Tier 3: CHRO + employment counsel; union consultation where required

#### Run-state obligations

- Monitoring policy reviewed annually and re-disclosed on change
- Contractor engagements reviewed annually for classification drift
- Training completion tracked; recertification cycles
- Post-change employee feedback and grievance tracking

### 10. Enterprise Risk and Operational Resilience

Owner: Chief Risk Officer, with the Business Continuity Management lead. Triggered when a project supports a critical business service, creates a new dependency or single point of failure, or introduces risk outside the company's risk appetite.

#### Domain taxonomy

- res.supports_critical_service (boolean): supports a service on the critical business services list
- res.recovery_targets (enum): RTO/RPO tier required (e.g., under 4 hours, same day, next business day, best effort)
- res.single_point_of_failure (boolean): one system, vendor, site or person with no fallback
- res.material_outsourcing (boolean): a third party performs a function the business depends on
- res.new_risk_category (boolean): risk type not already in the enterprise risk register
- res.risk_appetite_impact (enum): within appetite, near limit, exceeds appetite
- res.dependencies (multi-enum): internal systems, vendors, cloud regions, facilities, key people
- res.risk_tier (derived): low, medium, high

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| ERM-01 | res.supports_critical_service = true OR sec.system_criticality in high, critical | Tier 2: business impact analysis, BCP and DR plan, RTO/RPO agreed, resilience test before launch |
| ERM-02 | res.material_outsourcing = true OR prc.criticality = high | Tier 2: outsourcing risk assessment, concentration check, exit plan (with PRC-05); for federally regulated financial institutions, OSFI Guideline B-10 |
| ERM-03 | res.single_point_of_failure = true | Mitigation plan or a dated risk acceptance by the business owner |
| ERM-04 | res.new_risk_category = true OR res.risk_appetite_impact in near limit, exceeds appetite | Tier 3: enterprise risk assessment, risk register entry, CRO review |
| ERM-05 | geo.federally_regulated = true AND res.supports_critical_service = true | Tier 3: map to critical operations and impact tolerances under OSFI Guideline E-21 (operational resilience) |
| ERM-06 | fin.total_cost >= $1M OR proj.type = M&A | Enterprise risk summary for the investment committee |

#### Policies and standards

Enterprise Risk Management Framework, Risk Appetite Statement, Business Continuity Management Policy, IT Disaster Recovery Standard, Outsourcing and Third-Party Risk Policy (shared with PRC), Incident and Crisis Management Plan, Scenario Testing Standard. Frameworks: ISO 31000, ISO 22301, COSO ERM; OSFI E-21 and B-10 where federally regulated.

#### Approval chain

- Tier 1: business owner attestation
- Tier 2: BCM lead or operational risk manager
- Tier 3: CRO (risk committee for anything that exceeds appetite)

#### Run-state obligations

- BCP and DR tests at least annually for critical services; results and fixes tracked
- Business impact analysis refreshed every 2 years or on material change
- Risk register entries and key risk indicators reviewed quarterly
- Risk acceptances expire after 12 months
- Scenario exercises for critical services on a set cycle

### 11. Data Governance

Owner: Chief Data Officer, supported by a data governance council and domain data owners and stewards. Triggered when a project brings in new data, reuses data for a new purpose, shares data outside the company, or produces data others will rely on. Privacy covers personal information rules; Data Governance covers ownership, quality, lineage and sharing for all data.

#### Domain taxonomy

- dg.new_data_source (boolean): data not already in the data catalog
- dg.data_domains (multi-enum): customer, product, finance, employee, operations, third-party, public
- dg.critical_data_elements (boolean): feeds financial, regulatory or management reporting, or automated decisions
- dg.external_sharing (boolean): data shared with or received from a third party
- dg.master_data_change (boolean): creates or changes master or reference data
- dg.data_product (boolean): publishes a data set others in the company will reuse
- dg.deidentified_claim (boolean): data is described as anonymized or de-identified
- dg.lineage_known (boolean): source and transformations documented

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| DG-01 | dg.new_data_source = true OR data.new_use_of_existing = true | Tier 1: register in data catalog with a named owner and steward; classification confirmed (feeds data.classification) |
| DG-02 | dg.external_sharing = true | Tier 2: data sharing agreement (with LGL-01), purpose and use limits, onward-sharing rules; triggers PRV-09 if personal information |
| DG-03 | dg.critical_data_elements = true OR fin.financial_reporting_system = true | Tier 2: data quality rules, lineage documented, reconciliation controls |
| DG-04 | dg.master_data_change = true | Tier 2: master data review by the data domain owner |
| DG-05 | ai.uses_ai = true AND aig.training_data_source contains company data OR customer data | Data set documentation (provenance, rights, known gaps and bias), paired with AIG-07 |
| DG-06 | dg.deidentified_claim = true | Tier 2: re-identification risk review with Privacy (Quebec Law 25 sets specific anonymization requirements) |
| DG-07 | dg.data_product = true | Data product contract: owner, quality targets, refresh schedule, access policy |

#### Policies and standards

Data Governance Policy, Data Ownership and Stewardship Standard, Data Classification Standard (shared with SEC), Data Quality Standard, Metadata and Data Catalog Standard, Data Sharing Standard and agreement templates, Master Data Management Standard, De-identification Standard. Frameworks: DAMA-DMBOK as reference.

#### Approval chain

- Tier 1: data steward attestation
- Tier 2: domain data owner
- Tier 3: CDO or data governance council (e.g., new external data sharing of customer data at scale)

#### Run-state obligations

- Catalog entries and owners reviewed annually
- Data quality monitoring on critical data elements, with breaches routed to the owner
- Access to data products reviewed periodically
- Data sharing agreements tracked for renewal and termination (data return or destruction)

### 12. Enterprise Architecture

Owner: Chief Architect, through the Architecture Review Board (ARB). Triggered when a project introduces new technology, changes a core platform, duplicates an existing capability or departs from approved hosting and integration patterns.

#### Domain taxonomy

- ea.new_technology (boolean): technology not on the approved technology list
- ea.duplicate_capability (boolean): overlaps a capability an existing system already provides
- ea.core_platform_change (boolean): changes a designated core platform (e.g., ERP, CRM, identity, data platform)
- ea.approved_pattern (boolean): uses an approved hosting, integration and AI pattern
- ea.integration_count (number band): 0, 1-3, 4-10, more than 10 integrations
- ea.end_of_life_dependency (boolean): depends on technology at or near end of support
- ea.build_or_buy (enum): build, buy, configure existing, extend existing
- ea.replaces_system (boolean): retires or replaces an existing application

#### Applicability rules

| Rule | Triggers when | Result |
|---|---|---|
| EA-01 | sec.new_code = true OR vendor.new = true AND tech.hosting = vendor SaaS | Tier 1: architecture checklist; application recorded in the portfolio and capability map |
| EA-02 | ea.new_technology = true OR ea.approved_pattern = false | Tier 2: ARB review; technology decision recorded (adopt, trial, hold) |
| EA-03 | ea.duplicate_capability = true | Tier 2: rationalization review: reuse the existing capability or justify the duplicate (triggers FIN-02 business case) |
| EA-04 | ea.core_platform_change = true OR fin.total_cost >= $1M | Tier 3: full ARB review against target-state architecture |
| EA-05 | ai.uses_ai = true AND ai.model_source in third-party API, open-weight | Approved AI platform pattern (e.g., model access through the company's AI gateway), with AIG and SEC |
| EA-06 | ea.integration_count >= 4-10 OR sec.integrations contains external APIs | Integration and API standards review |
| EA-07 | ea.end_of_life_dependency = true | Technology lifecycle plan with an upgrade or exit date |
| EA-08 | ea.replaces_system = true | Decommissioning plan for the old system (paired with REC-04 and SEC access removal) |

#### Policies and standards

Architecture Principles, Approved Technology List (technology radar), Cloud and Hosting Patterns, Integration and API Standards, AI Platform Pattern, Application Portfolio Management Standard, Technology Lifecycle Standard. Frameworks: TOGAF as reference.

#### Approval chain

- Tier 1: solution architect attestation
- Tier 2: domain architect or ARB delegate
- Tier 3: full ARB, chaired by the Chief Architect (CTO for exceptions to architecture principles)

#### Run-state obligations

- Application portfolio and CMDB records kept current
- Technology lifecycle review annually; end-of-support dates tracked
- Replaced systems decommissioned by their planned date, with status in the admin dashboard
- Architecture exceptions and technical debt recorded with expiry dates

## Cross-domain trigger map

Domains feed each other through derived attributes and explicit cross-triggers. The engine resolves these automatically; this table is the human-readable view each SME should review for gaps.

| When this fires | It also triggers | Why |
|---|---|---|
| PRV-02 (high-risk personal info) | SEC-02 threat model; REC-06 retention alignment | Sensitive data needs stronger controls and minimized retention |
| PRV-09 (vendor processes personal info) | PRC-03 TPRM; LGL-01 DPA in contract | Data processing agreement and vendor assessment |
| PRV-04 (cross-border transfer) | PRC-06 location review; LGL-01 transfer clauses | Transfer mechanism must be in the contract |
| SEC-06 (vendor data access) | PRC-03 or PRC-04 | Security evidence collected through TPRM |
| SEC-04 (payment flow) | FIN-04 revenue review; LGL-02 customer terms | Payments imply billing and terms |
| LGL-05 (third-party/open-source content) | SEC-07 dependency scanning | Licence and vulnerability checks share an SBOM |
| PRC-01 (new vendor) | FIN-01 budget check; LGL-01 contract | Every purchase needs budget and a contract |
| PRC-07 (AI vendor) | AIG-01 inventory; LGL-01 AI terms; PRV-09 if personal info | Vendor AI terms on training, retention, output IP |
| AIG-03 (significant-impact AI) | PRV-02 PIA; RC-01 regulatory; SEC-08 AI security | Consequential decisions need privacy, legal and security review |
| AIG-08 / HR-07 (AI in hiring or HR) | Each other; PRV-02 | Employment decisions are a high-risk AI category |
| HR-02 (employee monitoring) | PRV-02; REC-01 | Monitoring data is personal information and a record |
| RC-10 (vendor UI component) | PRC-01 (ACR required in RFP) | Accessibility requirements go into procurement |
| REC-02 (new system of record) | SEC-01; PRV-01 if personal info | New repository needs security and privacy baseline |
| FIN-07 (financial reporting system) | SEC-05 IAM review; REC-05 | ICFR IT general controls and regulatory retention |
| Any project at proj.stage = retiring | REC-04; SEC (access revocation); PRC (vendor offboarding) | Decommissioning is a governed event too |
| ERM-01 (critical business service) | SEC-03 security assessment; PRC-05 exit plan for critical vendors | A critical service needs strong controls and a way out of any vendor |
| PRC-05 (critical vendor) | ERM-02 outsourcing risk | Critical vendors are an operational resilience question too |
| DG-02 (external data sharing) | LGL-01 data sharing agreement; PRV-09 if personal information | Sharing terms belong in a contract |
| DG-05 (AI training data) | AIG-07 data rights review | Training data needs both rights and documentation |
| EA-02 (new technology) | SEC-02 threat model; PRC-01 if a vendor supplies it | New technology brings new attack surface and often a new vendor |
| EA-03 (duplicate capability) | FIN-02 business case | A duplicate system needs a cost justification |
| EA-08 (system replaced) | REC-04 decommissioning; SEC access removal | Retiring a system is a governed event |

## Integrations

The platform orchestrates; specialist systems do the specialist work. Integrations are prioritized by how many manual steps they remove.

| System | Direction | Purpose | Phase |
|---|---|---|---|
| SSO / identity provider | In | Authentication, groups, roles | MVP |
| HRIS (org chart) | In | Resolve approvers by role and business unit; delegation | MVP |
| Slack / Microsoft Teams | Out | Notifications, approval actions, agent access in chat | MVP |
| Email | Out | Notifications and digests | MVP |
| Policy repository (SharePoint, Confluence, etc.) | In | Policy text and versions for agent retrieval and citations | MVP |
| Work management (Jira, ServiceNow, Asana) | Both | Create requirement tasks in teams' own tools; sync status | Phase 2 |
| Procurement / P2P (Coupa, Ariba, etc.) | Both | Vendor records, purchase requests, spend | Phase 2 |
| TPRM / GRC platform | Both | Vendor assessments, control library, risk register | Phase 2 |
| Contract lifecycle management | Both | Contract requests, obligation tracking | Phase 2 |
| Data catalog / CMDB | In | Cross-check data categories and systems against answers | Phase 2 |
| CI/CD and security scanners | In | Evidence of scans and test results | Phase 3 |
| E-signature | Out | Formal sign-off where required | Phase 3 |
| BI / data warehouse | Out | Portfolio analytics | Phase 3 |

All integrations use a published API and event stream (project.created, attribute.changed, requirement.completed, approval.decided, rule.published) so other tools can subscribe.

## Security, privacy and privilege of the platform itself

The platform will hold the company's most sensitive plans (unannounced products, M&A, reorganizations, legal advice), so it must meet its own Tier 3 bar.
- Access control. Projects are visible to the requester, their team, triggered SMEs and approvers. A project can be flagged confidential (e.g., M&A, workforce reduction), which limits visibility to named people and hides it from portfolio views.
- Privilege. Legal advice recorded in the platform is labelled "Privileged and Confidential, prepared at the request of counsel", stored in a separate privileged-notes field, excluded from general exports and analytics, and visible only to Legal and named recipients. Open question for Legal: whether agent conversations with in-house counsel's policy content can attract privilege (assume not).
- Audit log. Append-only, tamper-evident log of every attribute change, rule evaluation, agent extraction, approval and override.
- AI data handling. The LLM provider must not train on company data; zero or limited data retention; data residency in Canada where available; prompts and outputs logged internally.
- Retention. Project records retained per the Records Schedule (proposed: life of project plus 7 years); agent transcripts retained for 2 years unless attached to a decision record.
- Personal information. The platform itself is a project that goes through its own intake (dogfooding): PRV, SEC, AIG, REC all trigger.

## Reporting, audit and evidence

Reporting works at two levels from the same data. Global reporting gives governance, risk and executives one view across every domain. Per-domain reporting gives each domain owner the view of their own area. Every figure is computed from project records, rule evaluations and the audit log, never maintained by hand, and every report respects project confidentiality (confidential projects count in totals but are hidden from lists unless the viewer has access).

### Global reporting

Audience: governance admin, executive sponsor, risk committee, internal audit.

| Report | What it shows | Cadence |
|---|---|---|
| Portfolio overview | Projects by stage, business unit, province and overall risk tier; new intakes this period | Live; monthly snapshot |
| Cycle time | Median and 90th-percentile days from intake to each gate and to launch, by business unit and tier | Live; monthly trend |
| Bottlenecks | Domains, approval steps and approvers holding the most projects; SLA breaches | Live |
| Cross-domain coverage | How often each domain triggers, which domains trigger together, projects touching 5+ domains | Monthly |
| Exceptions and risk acceptances | Open exceptions by domain and owner, expiring in the next 30/60/90 days | Live |
| Run-state compliance | Obligations due, overdue and completed across all live projects | Live; quarterly to risk committee |
| Missed and late reviews | Reviews started after build, rule changes that newly triggered live projects | Monthly |
| Agent quality | Questions per intake, requester corrections to extracted facts, unknown-answer rate, policy Q&A routed to SMEs | Monthly |
| Evidence pack | Everything for one project: facts, rule versions, requirements and evidence, approvals, exceptions | On demand |

Regulatory and inventory outputs (generated, not maintained by hand): record of processing activities, AI system inventory with risk tiers, vendor inventory with risk tiers and data access, systems of record and their retention settings, accessibility conformance of public properties, and a provincial footprint (which provinces each live project touches).

### Per-domain reporting

Audience: each domain owner and their team. Every domain gets the same standard set, filtered to its own rules and projects, and the owner can save custom views.

| Report | What it shows |
|---|---|
| Domain queue | Projects awaiting this domain's review or approval, age, SLA status, assigned reviewer |
| Trigger analysis | How often each rule fires, by tier and province; rules that fire on almost everything or nothing |
| Requirement status | Open, completed and overdue requirements by project and stage |
| Waivers and exceptions | Exceptions granted in this domain, who accepted the risk, expiry dates, repeat waivers of the same requirement |
| Run-state obligations | This domain's recurring obligations due and overdue (e.g., PIA refreshes, vendor reassessments, access recertifications) |
| Rule health | Rules overdue for annual review, draft changes awaiting review, back-test results, questions often answered "unknown" |
| Reviewer workload | Reviews per person, time spent per tier, capacity versus demand |
| Domain inventory | The domain's regulatory output, such as the RoPA for Privacy or the AI inventory for AI Governance |

Domain-specific metrics owners can add (examples):
- Privacy: PIAs completed versus required, Quebec transfer assessments, data subject request readiness
- Security: projects with open high-risk findings, pen tests due, risk acceptances expiring
- Procurement and Vendor Risk: vendors by risk tier, reassessments overdue, critical vendors without an exit plan
- AI Governance: AI systems by impact level, monitoring results, AI incidents
- Finance: committed spend versus budget, projects without a business case review
- Records: systems without mapped retention, deletion jobs failing verification
- Regulatory and Compliance: licences and filings due, accessibility conformance of public properties, open exceptions
- HR: projects affecting employees by province, monitoring policies due for re-disclosure
- Legal: non-standard contracts approved, claims needing re-substantiation

### Delivery and export

- Scheduled email or Slack/Teams digests per audience (weekly for domain owners, monthly for executives).
- Export any report to CSV, Excel or PDF; read-only API and data-warehouse feed for BI tools.
- Point-in-time snapshots for audit, kept per the records schedule.

### Admin dashboard

The admin dashboard is the governance team's control centre: one place to track every project's status, see what is stuck and why, act on it, and run the platform. Access is role-based: governance admins see everything (subject to confidential-project restrictions), domain owners see the same views filtered to their domain, and executives get a read-only summary.

#### Project tracker

A filterable list of every project, with saved views (e.g., "Tier 3 launching this quarter", "Quebec projects with open privacy items").

| Column | Contents |
|---|---|
| Project | Name, business unit, requester, sponsor |
| Stage | Idea, discovery, design, build, pre-launch, live, retiring |
| Status | On track, at risk (a gating item is near its SLA or due date), blocked (a gate cannot clear), on hold, launched, retired |
| Domains | Triggered domains with each one's tier and state (not started, in review, approved, exception, done) |
| Next gate | Gate name, target date and the open items that block it |
| Open items | Count of open requirements and approvals; oldest item's age |
| Risk tier | Highest tier across domains |
| Provinces | Provinces in scope |
| Last activity | Most recent change, and by whom |

Views: table, board by stage, and calendar of upcoming gates and launches. Clicking a project opens its record: fact sheet, requirements by stage, approval chain progress, exceptions, obligations, documents and full history.

#### Status and alerts

- Status is computed, not typed: the platform sets "at risk" and "blocked" from SLA timers, due dates and gate rules, and shows the reason.
- Alert feed: SLA breaches, overdue obligations, exceptions about to expire, projects whose facts changed after approval, rule changes affecting live projects, and intakes with unresolved inconsistencies flagged by the agent.
- Bottleneck panel: approvers and domains with the longest queues and the most breaches.

#### Admin actions

- Reassign a reviewer or approver; set or end a delegation.
- Escalate an item or nudge its owner (sends a notification through email or Slack/Teams).
- Change a project's stage, place it on hold, or mark it confidential.
- Re-run rule evaluation for a project after a fact correction.
- Bulk actions on filtered lists (e.g., nudge all owners with overdue obligations).
- Every admin action is logged with who, when and why; admins cannot approve on a domain's behalf or waive requirements.

#### Platform administration

- Domain registry: onboard a domain, set its owner and backup owner, view its health score.
- Taxonomy manager: approve proposed attributes, deprecate old ones, see which rules use each attribute.
- Rule change queue: rule changes awaiting governance sign-off, with impact analysis on live and in-flight projects.
- Users and roles: role assignment, synced from SSO and HRIS.
- Integration health: status of each connected system and failed syncs.
- Usage: intakes started and completed, drop-off points in the intake conversation, active users.

#### Phasing

- MVP: project tracker, computed status, alert feed and core admin actions for the four pilot domains; per-domain queue and requirement status reports.
- Phase 2: full global and per-domain report sets, saved views, scheduled digests, platform administration screens.
- Phase 3: regulatory and inventory outputs, BI feed, custom domain metrics.

## Additional capabilities

Five capabilities extend the core platform. Together they cut repeat work for reviewers and keep approvals current after launch.

### 1. Reuse earlier approvals

Many projects share a vendor, platform or AI model that has already been assessed. Approved assessments attach to that shared component, so later projects inherit them instead of repeating the review.
- Reusable components. Vendors, platforms, AI models, data sets and shared services each get a record holding their approved assessments (e.g., TPRM assessment, threat model, model card and AIA), the facts those assessments relied on, the approver and an expiry date.
- Matching at intake. When the agent identifies a known component, it shows the existing approvals and the conditions attached to them. The requester confirms the component is used the same way.
- Fact comparison. The engine compares the new project's facts with the facts the original approval relied on. If they match within the approval's scope (same data categories or less sensitive, same provinces or fewer, same use), the requirement is marked satisfied by reuse. If anything goes beyond that scope (new data category, new province, higher volume, new use), only the changed part goes to review.
- Expiry and conditions. Expired approvals cannot be reused; the first project to need one triggers reassessment. Conditions on the original approval (e.g., "no customer data") become requirements on every project that reuses it.
- Owner control. Each domain owner decides which requirements can be satisfied by reuse and for how long (e.g., vendor security assessment: 12 months; PIA: never reused across purposes).
- Ripple effect. If a component's approval is withdrawn or a reassessment finds a problem, every project relying on it is flagged in the admin dashboard.
- Reporting. Global and per-domain reports show reviews avoided through reuse and components with approvals about to expire.

### 2. Reviewer assistant

The intake agent serves requesters; the reviewer assistant serves domain specialists. It prepares the work, and the specialist decides.
- Project brief per domain. A one-page summary for each reviewer: confirmed facts relevant to their domain, the rules that fired and why, related projects and reusable approvals, and open questions.
- Draft assessments. First drafts of the PIA, Algorithmic Impact Assessment, threat model, vendor risk summary or accessibility plan, filled from confirmed facts and the domain's templates. Every statement in a draft links to the fact or document it came from; gaps are marked for the reviewer, never filled in by guesswork.
- What's unusual. Flags anything that differs from similar past projects (e.g., more data categories than usual, a province not seen before, an unusual vendor location) and any inconsistency between documents and answers.
- Questions back to the requester. The reviewer can have the assistant draft follow-up questions, sent through the project thread.
- Guardrails. Same as the intake agent: the assistant cannot approve, reject, waive or change rule outcomes. Drafts are labelled as drafts until a named reviewer edits and signs them. Legal analysis it drafts is marked privileged where counsel requested it.
- Measurement. Track reviewer time per tier before and after, how much of each draft reviewers keep, and corrections made, as an input to the agent evaluation set.

### 3. Regulatory change tracking

Legal and regulatory changes flow into each domain owner's rule review queue, so the rules stay current across the ten provinces and federal law.
- Sources. A regulatory update feed (subscription service or Legal's own monitoring), plus manual entries by any domain owner or counsel.
- Change records. Each change has the law or guidance affected, jurisdiction, status (proposed, passed, in force), effective date and a short summary. Example watch items today: federal Bill C-36 (introduced, not in force) and Ontario's Consumer Protection Act, 2023 (passed, awaiting proclamation).
- Mapping to rules. Each change is linked to the rules, requirements and policies it touches. The agent suggests likely matches; the domain owner confirms them.
- Impact list. For each change, the platform lists live and in-flight projects that would be affected, using the same impact analysis as rule changes (e.g., all live projects with Ontario consumers for the new Consumer Protection Act).
- Workflow. The change enters the owning domain's rule review queue with a due date tied to the effective date. The owner drafts the rule change (the rule maintenance assistant can propose a first draft), back-tests it, and publishes it through normal change control, choosing whether it applies to new projects only or to existing ones too.
- Reporting. A regulatory change register in global reporting: open changes, owners, days to effective date, and rules not yet updated.

### 4. Annual attestations

Once per year, each live project's owner confirms its facts are still true. This catches drift that no one reported.
- Scope. Every project in the live stage, on the anniversary of launch (or in a single annual campaign, at the governance admin's choice).
- What is asked. Only the facts that drove the project's rules, phrased as confirmations: "Still no health data?", "Still only Ontario and Quebec users?", "Same vendor and hosting location?"
- Outcomes. No change: recorded as an attestation with the owner's name and date. Any changed answer: the engine re-evaluates the project and opens review only for newly triggered or changed requirements.
- Follow-up. Reminders at 14 and 30 days, then escalation to the project's executive sponsor; overdue attestations show in the admin dashboard and global reporting.
- Ownership changes. If the project owner has left (from HRIS), the attestation goes to their manager to reassign first.
- Evidence. Attestations are part of the project's evidence pack.

### 5. Rule maintenance assistant

Domain owners keep twelve rule packs current across ten provinces and federal law. The rule maintenance assistant does the drafting, testing and housekeeping so owners can focus on the judgment calls. It proposes; the owner decides and publishes through normal change control.
- Draft rules from policy text. The owner points the assistant at a policy section, standard or regulatory change record. It proposes the applicability rule, requirements and approval chain changes in the rule language, with a plain-English rendering and a citation to the exact source section for each condition.
- Tests and back-tests. For every draft it writes test cases (projects that should and should not trigger), runs the back-test against past projects, and summarizes the impact: which projects would newly trigger, which would stop, and how reviewer workload would change.
- Gap and conflict detection. It scans the domain for policy requirements with no rule, rules with no source policy, rules that fire on almost everything or nothing, attributes no rule uses, and overlaps or contradictions with other domains' rules. Findings go to the owner's rule review queue.
- Regulatory change follow-through. For each item in the regulatory change queue, it proposes the specific rule edits and the list of affected live projects, so the owner starts from a draft rather than a blank page.
- Annual review preparation. Before each rule's annual review, it summarizes trigger rates, waivers granted, reviewer comments and requester confusion (questions often answered "unknown"), and recommends keep, change or retire.
- New attributes. When a rule needs a fact the shared taxonomy lacks, it drafts the attribute (type, allowed values, intake question) and routes it to the governance admin's taxonomy change control.
- Plain-language explanations. It drafts the rationale requesters see for each rule and the change note for the rule change log.

#### Guardrails

- Suggestions only: the assistant cannot publish, activate or retire a rule, or change a taxonomy attribute.
- Every draft goes through the same path as a human draft: tests, back-test, peer review by a second SME, then publish (with governance admin sign-off for material changes).
- It cites only the company's policy library and regulatory change records. It never states a legal requirement without a source, and it flags uncertainty instead of guessing.
- Drafts are labelled as AI-assisted until the owner edits and approves them; the owner remains accountable for the rule.
- All suggestions, accepted or rejected, are logged with the rule's history.

Measurement: days from a regulatory change's effective date to the updated rule being live, share of drafts accepted with minor edits, gaps found and closed, and rules overdue for annual review (target: none).

### Phasing

- Scale phase: annual attestations, regulatory change tracking, approval reuse for vendors.
- Optimize phase: approval reuse for platforms, AI models and data sets; reviewer assistant (piloted first with Privacy and Vendor Risk, then extended); rule maintenance assistant (gap and conflict detection first, then rule drafting).

## Non-functional requirements

| Area | Requirement |
|---|---|
| Explainability | Every triggered requirement links to the rule, version, and the attribute values that caused it |
| Determinism | Re-running evaluation on a frozen snapshot gives identical results |
| Availability | 99.9% monthly for intake and approvals; rules engine degrades to "trigger all Tier 2 reviews" if unavailable, never to "trigger none" |
| Performance | Rule evaluation under 2 seconds for 1,000 active rules; agent first response under 5 seconds |
| Scale | 5,000 projects per year; 50 domains; 2,000 rules |
| Multi-entity | Supports subsidiaries and jurisdictions with entity-specific rules and approvers |
| Localization | English and French UI and agent (required for Quebec users) |
| Accessibility | The platform itself meets WCAG 2.1 AA |
| Data residency | Canadian hosting by default |
| Interoperability | Rules and taxonomy exportable (JSON/YAML) to avoid lock-in |

## Risks, mitigations and open questions

| Risk | Mitigation |
|---|---|
| A required review is missed and the company relies on the tool | Unknowns default to trigger; gold-set testing with zero tolerance on key triggers; back-testing; requester attestation; tool positioned as aid, with domain owners accountable for rules |
| Requesters game answers to avoid reviews | Cross-checks against systems of record; agent flags inconsistencies with attached documents; random SME audit of Tier 1 projects; attestation on record |
| Over-triggering makes the tool a bottleneck | Tiering with self-service Tier 1; trigger-rate dashboards; SLA tracking; back-test before publishing |
| SMEs do not adopt or maintain rules | Executive mandate; start with domains that feel the most pain; health alerts; rules reviewed in quarterly governance forum |
| Rule sprawl and contradictions | Shared taxonomy; publish-time conflict checks; governance admin precedence decisions |
| Agent hallucination in policy Q&A | Retrieval-only answers with citations; "I don't know, routing to SME" behaviour; regular evaluation |
| Sensitive project leakage | Confidential flag, need-to-know access, privilege controls |
| Regulatory change outpaces rules | Regulatory watch feeds rule review queue; impact analysis on change |

### Open questions

- [ ] Who owns the shared attribute taxonomy: central Legal Ops, Risk, or a PMO?
- [ ] Build, buy, or extend an existing GRC platform? (See build-vs-buy analysis, to be written.)
- [ ] Which business unit pilots first?
- [ ] What is the threshold for a "material change" that re-triggers review?
- [ ] Is intake mandatory for all projects, or above a size threshold?
- [ ] Final dollar and volume thresholds for each domain's tiers

## Success metrics

Targets are proposals to confirm once discovery sets baselines.

| Metric | Target at 12 months |
|---|---|
| Projects entering through intake (in pilot scope) | 90% or more |
| Median kickoff-to-all-approvals time | 30% reduction from baseline |
| Reviews started after build began | Under 10% of projects |
| Missed-review incidents | Zero in pilot scope |
| Questions asked per intake | Under 25 (median) |
| Agent extraction accuracy on triggering attributes | 95% or more precision; 99% or more recall |
| Requester satisfaction (CSAT) | 4 out of 5 or higher |
| SME triage hours per project | 40% reduction |
| Rules reviewed within 12 months | 100% |

## Phasing and roadmap

The MVP proves the model on the four domains with the most cross-triggers before the remaining eight onboard; each phase starts only when its gate criteria are met, so these dates are planning assumptions, not commitments.

> The roadmap diagram is not included in this copy. See the phase table and gates in [the executive summary](governance-intake-platform-exec-summary.md#11-pilot-four-domains-by-mid-2027-then-scale-to-all-twelve).

## Appendix: worked example

Project: "Smart Support Assistant". A generative-AI chatbot on the public website answers customer billing questions, using customer account history. Built on a third-party LLM API, new vendor, $180k first-year cost. Users in Ontario, Quebec and Alberta. Launch target: Q2.

Facts the agent extracted from the brief, then the requester confirmed:
- data.personal_info = true; data.categories = contact, financial, behavioural; data.subject_volume = 100k-1M
- ai.uses_ai = true; ai.role = customer-facing assistant; ai.model_source = third-party API
- vendor.involved = true; vendor.new = true; vendor.data_access = confidential
- geo.user_jurisdictions = ON, QC, AB; geo.cross_border_transfer = true (vendor hosts outside Canada)
- tech.internet_exposed = true; ux.new_digital_interface = true; acc.audience_scope = public
- fin.total_cost = $25k-$250k; fin.budgeted = true; legal.new_contract = true
- rec.creates_records = true; rec.format includes email/chat

Engine result:

| Domain | Rules fired | Tier | Key requirements | Approver |
|---|---|---|---|---|
| Privacy | PRV-01, 02, 03, 04, 09 | 3 | PIA; Quebec transfer assessment; DPA; French notice | Privacy counsel, then CPO |
| Security | SEC-01, 02, 06, 08 | 2 | Threat model; pen test; AI security review | Security architect |
| AI Governance | AIG-01, 02 | 2 | AI inventory; AI disclosure to users; output safety testing | AI governance lead |
| Procurement | PRC-01, 02, 03, 07 | 3 | Vendor onboarding; 3 quotes; TPRM; AI vendor terms | Procurement + TPRM lead |
| Legal | LGL-01 | 2 | Vendor contract review | Commercial counsel |
| Finance | FIN-01 | 1 | Budget check | Budget owner |
| Records | REC-01, 03 | 1-2 | Chat transcript retention mapping | Records Manager |
| Regulatory and Compliance | RC-06, 07, 08 | 2 | French-language review for Quebec users; WCAG AA audit of chat widget | French-language and accessibility leads |
| HR | none | - | Not triggered | - |
| Data Governance | DG-01 | 1 | Catalog entry with owner for customer account data used by the assistant | Data steward |
| Enterprise Architecture | EA-01, EA-05 | 2 | Portfolio record; model access through the approved AI gateway pattern | Domain architect |
| Enterprise Risk and Resilience | none | - | Not triggered unless billing support is on the critical services list; the agent asks one follow-up | - |

Why HR did not trigger: hr.affects_employees = false. The agent asks one follow-up: "Will support agents' work or monitoring change?" A yes would add HR-01 and possibly HR-02.

Run state created: annual PIA refresh, monthly AI output monitoring, annual vendor reassessment, chat transcript deletion job, release-level accessibility scans, renewal alert 90 days before contract term ends.

## Sources

Provincial law references, checked November 25, 2026:
- Provincial laws that may apply instead of PIPEDA (Office of the Privacy Commissioner of Canada)
- Privacy Laws: Doing Business in Canada 2026 (McCarthy Tétrault)
- Implications of Bill C-36 for organizations (MLT Aikins)
- Alberta PIPA engagement (Government of Alberta)
- Changes to Alberta's Health Information Act (IAPP)
- Accessibility law in Canada: cross-country legislation (Ogletree Deakins)
- Written policy on electronic monitoring of employees (Government of Ontario)
- AI in hiring: Ontario job posting disclosure (Osler)
- Pay transparency laws in Canada: province-by-province guide (2026) (Employment Hero)
- Pay transparency in BC (Government of British Columbia)
- Pay equity laws in Canada (ADP Canada)
- Consumer Protection Act, 2023 (not yet in force) (Ontario e-Laws)
