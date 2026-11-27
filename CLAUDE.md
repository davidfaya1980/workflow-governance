# CLAUDE.md

Guidance for Claude when working in this folder.

## About this project

Enterprise Workflow Governance is a project folder (mostly documents, not an application codebase). It holds the policies, workflow definitions, decision records and research for governing workflows across the organization.

## Requirements and specifications

The system being built is the Governance Intake Platform. Its first-draft requirements and specs are indexed in `docs/source-artifacts.md`, with local copies of the design document (`docs/governance-intake-platform-design-doc.md`, the detailed spec and the source to follow when the two differ) and the executive summary deck (`docs/governance-intake-platform-exec-summary.md`). Read these before writing policies, workflows or decisions, and keep the index up to date when artifacts are added or revised. Exported copies should match their source artifact; don't edit them by hand.

Core design principles to respect in everything written here:
- The AI never decides. Deterministic, versioned rules decide which domains apply; named people approve.
- Every requirement traces back to a rule and the facts that triggered it.
- Unknown or ambiguous answers trigger review (over-review, never a silent miss).
- Each fact is asked once, in one shared taxonomy. Attributes are additive; a meaning change creates a new attribute.
- Rules are deterministic and versioned; same facts plus same rule versions give the same result.
- The platform itself must meet its own Tier 3 bar (Canadian residency, privilege controls, tamper-evident audit log).

Key terms: attributes (typed facts, IDs like `data.personal_info`), rules (IDs like `PRV-02`), 12 domains (each a "domain pack" owned by its domain owner, with taxonomy, rules, policies, approval chain and run-state obligations), review tiers (1 notify, 2 expert review, 3 escalate), run state (obligations that continue after launch), project gates (design, pre-launch), programme phases (Discovery, MVP pilot, Scale, Optimize) separated by Gates 1 to 3. Use the design doc's terms and IDs rather than inventing new ones.

## Layout

- `docs/`: overview, scope, roadmap and reference material
- `docs/decisions/`: decision records, named `NNNN-short-title.md`, numbered in order
- `policies/`: governance policies and standards
- `workflows/`: workflow definitions, process maps, RACI charts, SOPs
- `templates/`: reusable templates; start new documents from these when one fits
- `research/`: background research and evaluations
- `notes/`: meeting and working notes, named `YYYY-MM-DD-topic.md`
- `scripts/`: helper scripts

## Conventions

- Write in Markdown. Use lowercase, hyphenated file names.
- Put a short purpose statement and an owner/status/last-updated line at the top of each policy, workflow and decision document.
- Keep one topic per file and link related files with relative links.
- Record significant choices as a decision record in `docs/decisions/` rather than burying them in notes.
- Do not store secrets, credentials, or confidential personal data in this repo.

## Rules for Claude

- Ask David before editing existing code or config, committing, pushing, merging, or opening a pull request.
- Creating new draft documents in the folders above is fine; say what was added.
- Keep the README layout table in sync when adding or renaming top-level folders.
- The remote is `origin` (GitHub: `davidfaya1980/workflow-governance`), default branch `main`.
