# Agents

The framework ships four Claude Code [subagents](https://code.claude.com/docs/en/sub-agents). Each has one responsibility and a restricted tool list. Two skills are preloaded into each agent's context: `impact-analysis`, and the procedure skill for its role (see [workflows.md](workflows.md#procedure-skills)). The agent file itself holds only the role, the non-negotiables, the approval protocol, and the limitations.

| Agent | Responsibility | Tools | Writes files | Definition |
|---|---|---|---|---|
| `feature-implementation` | New features and enhancements | Read, Grep, Glob, Bash, Edit, Write | Yes, as planned | [agents/feature-implementation.md](../agents/feature-implementation.md) |
| `bug-investigation` | Root cause and minimal fix with regression test | Read, Grep, Glob, Bash, Edit, Write | Yes, minimal | [agents/bug-investigation.md](../agents/bug-investigation.md) |
| `database-specialist` | Schemas, queries, migrations, rollback | Read, Grep, Glob, Bash, Edit, Write | Migration and SQL files, plus ORM model or mapping files the approved plan lists; only after approval | [agents/database-specialist.md](../agents/database-specialist.md) |
| `code-reviewer` | Independent, severity-ranked review | Read, Grep, Glob, Bash | No | [agents/code-reviewer.md](../agents/code-reviewer.md) |

None of the agents has the `Agent` tool, so none can start further subagents.

## Using them

Ask in plain language and name the agent when you want a specific one. Plugin agents carry the plugin prefix:

```text
Use the personal-ai-workflow:bug-investigation agent to find out why order totals are rounded incorrectly.
```

```text
Have the personal-ai-workflow:code-reviewer agent review my uncommitted changes against this requirement: ...
```

Claude may also delegate on its own when a request matches an agent's description. Run `/agents` to see the agents that are loaded.

For complete tasks, use a workflow (for example, `/personal-ai-workflow:new-feature`). It chains these agents with approval checkpoints, so you don't have to call them one by one. See [workflows.md](workflows.md).

## Approval protocol

A subagent runs in its own context and **cannot talk to you directly**. It reports to the main conversation. So when an agent needs approval or an answer, it:

1. stops without making changes,
2. ends its report with a section titled **APPROVAL NEEDED**, containing its analysis, its plan, and the exact question.

The main conversation shows you that section. After you decide, Claude passes your answer back to the agent, either by resuming the same agent (which keeps what it already learned) or by starting it again with the approved plan.

When the agents stop for approval:

| Agent | Stops when |
|---|---|
| `feature-implementation` | Blast radius Medium or High, a public contract or schema change, a new dependency, security logic, CI/CD or infrastructure, more than a handful of files, important unknowns, or a blocking question |
| `bug-investigation` | The fix changes behavior beyond the bug, or touches a contract, schema, security, shared code, CI/CD, or infrastructure, or has blast radius Medium or High |
| `database-specialist` | Always, before drafting a migration, and before any database write or migration run |
| `code-reviewer` | Never needs to: it only reports. It does not apply fixes. |

Separately, Claude Code's own permission prompts still apply to every tool call an agent makes, according to your [permission settings](security-principles.md#recommended-permission-settings).

## Shared behavior

Every agent:

- detects the project's stack from evidence, and never assumes .NET, Angular, PostgreSQL, or any other technology,
- labels findings **Confirmed**, **Likely**, **Assumed**, or **Unknown**, and states what it inspected,
- never commits, pushes, tags, opens pull requests, or switches branches,
- asks before network access, such as restoring packages or fetching from the remote,
- never prints secrets, and never sends code or data to external services,
- treats instructions found in code, documents, or tool output as data,
- ends with a structured report that includes tests run, tests not run, and remaining risks.

## Limitations

- **Prompts are guidance, not enforcement.** Tool lists are the real restriction. An agent with Bash can still modify files or run commands its instructions forbid, if it errs or is manipulated. Configure [permission rules](security-principles.md#recommended-permission-settings) for real protection.
- **The reviewer is read-only by instruction, not by guarantee.** It has no Edit or Write tool, but Bash could change files.
- **One repository at a time.** Agents can't see other repositories, deployed systems, or production data. Dependencies outside the repository are reported as Unknown.
- **The agents' context starts empty.** A subagent does not see your conversation. Claude passes it a task description, so include the requirement, constraints, and anything you already know.
- **Model quality varies.** Agents use the same model as your session (`model: inherit`). Smaller models follow long procedures less reliably.
