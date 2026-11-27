# Enterprise Workflow Governance

Working folder for the Enterprise Workflow Governance project: the policies, workflow definitions, decisions and supporting research for governing workflows across the organization.

## Requirements and specifications

The system being built is the **Governance Intake Platform**: one front door for governance review from idea to launch and run state. An intake agent gathers facts, owner-maintained versioned rules decide which of 12 governance domains apply, and named people approve.

- [docs/source-artifacts.md](docs/source-artifacts.md): index of the spec artifacts, with a summary, roadmap and open decisions
- [docs/governance-intake-platform-design-doc.md](docs/governance-intake-platform-design-doc.md): full design document (Draft v0.1), the detailed spec
- [docs/governance-intake-platform-exec-summary.md](docs/governance-intake-platform-exec-summary.md): executive summary deck (Draft v0.1), exported to Markdown

## Layout

| Folder | What goes here |
|---|---|
| `docs/` | Overview, scope, roadmap and other reference documents |
| `docs/decisions/` | Decision records, one file per decision (`NNNN-short-title.md`) |
| `policies/` | Governance policies and standards |
| `workflows/` | Workflow definitions, process maps, RACI charts and SOPs |
| `templates/` | Reusable templates (decision record, policy, workflow, change request) |
| `research/` | Background research, vendor and tool evaluations, benchmarks |
| `notes/` | Meeting notes and working notes (`YYYY-MM-DD-topic.md`) |
| `scripts/` | Helper scripts and automation, if any |

## Working with Claude

See [CLAUDE.md](CLAUDE.md) for the conventions Claude follows in this folder.
