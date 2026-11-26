# CLAUDE.md

Guidance for Claude when working in this folder.

## About this project

Enterprise Workflow Governance is a project folder (mostly documents, not an application codebase). It holds the policies, workflow definitions, decision records and research for governing workflows across the organization.

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
