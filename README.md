# Personal AI Workflow

A reusable, technology-aware AI development framework for [Claude Code](https://code.claude.com/docs/en/overview), built from specialized **agents**, reusable **skills**, and standard **development workflows**.

The goal is a consistent, safe, and transparent way of working with an AI coding assistant across different projects, with **impact analysis before change**, **human approval for risky operations**, and **independent review** after implementation.

> **Status: 0.1.0.** First complete version: four agents, five procedure skills, and four workflows. See [Getting started](docs/getting-started.md).

---

## Why this exists

AI coding assistants are most useful when they behave like a careful senior engineer: they read before writing, explain what they found, keep changes small, and ask before doing anything hard to undo. This framework writes those habits down once so they can be reused in every project.

It is deliberately:

- **Generic**: no employer, client, or project-specific content. Ever. See [Client Data Protection](docs/security-principles.md#client-data-protection-policy).
- **Stack-aware, not stack-imposing**: agents inspect the current project's technology first and adapt. There is particular depth for C#/.NET, ASP.NET Core, Angular, TypeScript/JavaScript, PostgreSQL, SQL Server, REST APIs, Docker, Git, and CI/CD, but none of these are assumed.
- **Small**: a clean foundation that can grow, not a large rule system.

## How it works

The repository is packaged as a **Claude Code plugin**. Claude Code discovers the plugin's agents and skills from their standard locations, so the same files work in any project without being copied into it.

| Building block | What it is | Where it lives | Claude Code mechanism |
|---|---|---|---|
| **Agents** | Specialized subagents with a narrow role, their own system prompt, and a restricted tool list | `agents/*.md` | [Subagents](https://code.claude.com/docs/en/sub-agents) |
| **Skills** | Reusable procedures (for example, impact analysis) that Claude loads when relevant or that you invoke by name | `skills/<name>/SKILL.md` | [Agent Skills](https://code.claude.com/docs/en/skills) |
| **Workflows** | End-to-end processes (new feature, bug fix, ...) that chain skills and agents with human checkpoints. You start them as slash commands. | `skills/<workflow>/SKILL.md` | Agent Skills, user-invoked ([details](docs/architecture.md#workflows)) |

See [docs/architecture.md](docs/architecture.md) for the design and the reasoning behind it.

## Components

### Agents

| Agent | Responsibility | Writes files |
|---|---|---|
| [`feature-implementation`](agents/feature-implementation.md) | Understands the requirement, analyzes impact, proposes a plan, implements incrementally with tests. | Yes |
| [`bug-investigation`](agents/bug-investigation.md) | Reproduces the problem, finds the root cause, applies the smallest safe fix, and adds a regression test. | Yes |
| [`database-specialist`](agents/database-specialist.md) | Analyzes schemas, queries, and migrations; drafts safe migrations with rollback plans; instructed never to run destructive operations without approval. | Migration, SQL, and plan-listed ORM model files, after approval |
| [`code-reviewer`](agents/code-reviewer.md) | Reviews changes independently; reports findings by severity with file references. | No |

How to use them, when they stop for approval, and their limitations: [docs/agents.md](docs/agents.md).

### Procedure skills

Each one is preloaded into its agent. You or Claude can also use it directly.

| Skill | Purpose | Preloaded into |
|---|---|---|
| [`impact-analysis`](skills/impact-analysis/SKILL.md) | The mandatory pre-change analysis: dependencies, blast radius, regressions, tests. Distinguishes confirmed facts from assumptions. | All agents |
| [`feature-implementation`](skills/feature-implementation/SKILL.md) | Requirement, plan, checkpoint, incremental implementation, tests. | `feature-implementation` |
| [`bug-fixing`](skills/bug-fixing/SKILL.md) | Root-cause investigation, minimal fix, regression test. | `bug-investigation` |
| [`database-analysis`](skills/database-analysis/SKILL.md) | Schema, integrity, locking, safe migrations, rollback, query review. | `database-specialist` |
| [`independent-review`](skills/independent-review/SKILL.md) | Severity-ranked, evidence-based review with verdict rules. | `code-reviewer` |

### Workflows

Each workflow is a skill that only you can start, by name. It runs the steps in order, delegates to the agents, and pauses for your approval at the checkpoints. It is instructed never to commit, push, or publish. That is an instruction, not a technical block; see [Prompts are not security boundaries](docs/security-principles.md#prompts-are-not-security-boundaries).

| Workflow | Invoke with | Flow |
|---|---|---|
| [`new-feature`](skills/new-feature/SKILL.md) | `/personal-ai-workflow:new-feature <requirement>` | requirements → impact analysis → plan → approval → implement → test → diff review → independent review → summary |
| [`bug-fix`](skills/bug-fix/SKILL.md) | `/personal-ai-workflow:bug-fix <bug report>` | investigate → root cause → impact → minimal fix → regression test → test → diff review → summary |
| [`database-change`](skills/database-change/SKILL.md) | `/personal-ai-workflow:database-change <change>` | inspect schema → integrity analysis → proposal → rollback plan → approval → migration → non-production validation → report |
| [`pull-request-review`](skills/pull-request-review/SKILL.md) | `/personal-ai-workflow:pull-request-review <PR or branch>` | requirement → full diff → regressions → tests → security and performance → findings by severity → assessment |

Checkpoints, examples, and how reviewer independence is preserved: [docs/workflows.md](docs/workflows.md).

## Core principles

1. **Modularity**: each agent and skill has one clearly defined responsibility.
2. **Reusability**: instructions are generic and independent of any client.
3. **Maintainability**: documentation is short, clear, and easy to update.
4. **Safety**: read before writing, prefer small changes, avoid regressions.
5. **Transparency**: explain findings, assumptions, and proposed changes; separate *confirmed* from *assumed*.
6. **Human oversight**: destructive or high-impact operations need explicit approval.
7. **Project independence**: no proprietary client information in this framework.

The full rules are in [docs/security-principles.md](docs/security-principles.md).

## Installation

**Requirements:** Claude Code with plugin support, and Git.

In Claude Code:

```text
/plugin install personal-ai-workflow --marketplace AsareelDadiouari/personal-ai-workflow
```

That single command uses the `--marketplace` option of `/plugin install`, which `claude plugin validate` documents for Claude Code 2.1.275 and later. On older versions, use two steps:

```text
/plugin marketplace add AsareelDadiouari/personal-ai-workflow
/plugin install personal-ai-workflow@personal-ai-workflow
```

Other options (local clone, one-session loading), how to check the installation, first runs, and troubleshooting: **[docs/getting-started.md](docs/getting-started.md)**.

**Cost:** the plugin adds about 1,150 tokens to each session (the skill and agent descriptions). Each skill or agent costs roughly 0.5k–1.5k more tokens when it runs. These are estimates from `claude plugin details`, measured on Claude Code 2.1.331.

**Strongly recommended:** add real permission rules to your Claude Code settings. Prompts are guidance, not enforcement. See [Recommended permission settings](docs/security-principles.md#recommended-permission-settings).

## Using it in an organization's project

Before using this framework (or any AI assistant) on an employer's or client's code, confirm that:

- the organization permits the AI provider and tool you are using to process its code, and
- your use complies with its security, privacy, and AI usage policies.

The framework contains only generic instructions. Project-specific context belongs in that project's own `CLAUDE.md`, inside the organization's authorized environment, and never in this repository.

## Repository layout

```
personal-ai-workflow/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Lets Claude Code install the plugin from this repository
├── CLAUDE.md                    # Instructions for working on THIS repository
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── SECURITY.md
├── agents/                      # Subagent definitions
├── skills/                      # One folder per skill (procedures and workflows), each with SKILL.md
└── docs/
    ├── getting-started.md
    ├── agents.md
    ├── workflows.md
    ├── architecture.md
    └── security-principles.md
```

## Roadmap

Version 0.1.0 is the first complete version (see [CHANGELOG.md](CHANGELOG.md)). Possible next steps, added only when a real need appears:

- Optional hooks that block clearly dangerous commands (a real control, not just an instruction).
- More procedure skills, for example dependency upgrades or performance investigation.

## License

[MIT](LICENSE) © 2026 AsareelDadiouari.

The license covers this framework only. Code you write in your own or your clients' projects while using it is not affected by it and does not become part of it.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
