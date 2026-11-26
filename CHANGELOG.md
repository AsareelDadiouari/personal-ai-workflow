# Changelog

All notable changes to this framework are listed here. Versions follow [semantic versioning](https://semver.org/): a breaking change to an agent's or skill's behavior or name raises the major version.

## 0.3.0 (2026-11-26)

Ticket-driven work.

- **New `ticket` workflow:** give it an Azure DevOps work item, a GitHub issue, or a Jira issue (an ID, a URL, or pasted text). It reads the ticket, classifies it as bug, feature, database change, or other, and runs the matching workflow. It asks you first when the classification is uncertain or conflicts with the ticket's type field.
- **New `ticket-intake` procedure skill**, with a tracker reference loaded only when needed: reference formats, CLI commands, type fields, and branch and commit linking conventions.
- `new-feature`, `bug-fix`, and `database-change` accept a ticket ID or URL directly, and warn when the ticket looks like it belongs to another workflow.
- Tickets are read only: never changed, never stored in memory, and treated as data. Ticket comments are drafted for you and posted only with your approval.

## 0.2.0 (2026-11-24)

Token efficiency.

- **Fast path** in `new-feature`, `bug-fix`, and `pull-request-review`: small, low-risk changes are handled in the main conversation instead of extra agents. `new-feature` still runs the independent review. `--full` forces the full path. `database-change` has no fast path.
- **`--review-model <model>`** runs the independent reviewer on another model, for example `sonnet`.
- **Lighter reviewer:** `code-reviewer` no longer preloads `impact-analysis`. `independent-review` now carries a compact evidence and blast-radius section, saving about 1.2k tokens per review. Its report adds a blast-radius line.
- **Shorter descriptions:** the per-session cost dropped from about 1,150 to about 590 tokens, as estimated by `claude plugin details`.
- **Measured** on a small benchmark feature (one run each): total cost per `new-feature` run fell from $0.76 to $0.44 (about −42%), and output tokens halved. Wall-clock time rose (55 s → 81 s) because the work ran in one conversation instead of parallel agent startups.

## 0.1.0 (2026-11-24)

First version.

- Four agents: `feature-implementation`, `bug-investigation`, `database-specialist`, `code-reviewer`.
- Five procedure skills: `impact-analysis`, `feature-implementation`, `bug-fixing`, `database-analysis`, `independent-review`.
- Four workflow skills, started only by the user: `new-feature`, `bug-fix`, `database-change`, `pull-request-review`.
- Security principles, client data protection policy, and recommended permission settings.
