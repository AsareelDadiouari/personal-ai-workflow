# Workflows and skills

The framework has two kinds of skills. See [architecture.md](architecture.md#workflows) for the reasoning behind the split.

| Kind | Started by | Runs in | Purpose |
|---|---|---|---|
| **Workflow skills** | You, by name only | The main conversation | Order of steps, which agent does each step, and approval checkpoints |
| **Procedure skills** | Preloaded into agents, or loaded by Claude when relevant | An agent, or the main conversation | How to do one task well |

## Workflows

Each workflow skill sets `disable-model-invocation: true`, so Claude never starts one on its own. You start it, with a description of the task:

```text
/personal-ai-workflow:new-feature Add CSV export to the orders list, limited to the current filter
/personal-ai-workflow:bug-fix Order totals show 10.00 instead of 9.99 for discounted items
/personal-ai-workflow:database-change Add a nullable shipped_at timestamp to orders, with an index
/personal-ai-workflow:pull-request-review feature/csv-export against main
```

The main conversation coordinates. It delegates the work to agents, checks their reports, re-runs tests, reads the diff, and **stops at each checkpoint (⏸) until you answer**. Workflows are instructed never to commit, push, or publish. Those steps are left to you, and your permission rules should back that up.

| Workflow | Agents used | Checkpoints (⏸) |
|---|---|---|
| [`new-feature`](../skills/new-feature/SKILL.md) | `feature-implementation`, then `code-reviewer` | Ambiguous requirements; plan approval when the change is substantial or risky; failing tests; unexpected files in the diff; which review findings to fix |
| [`bug-fix`](../skills/bug-fix/SKILL.md) | `bug-investigation`; `code-reviewer` when risky | Missing reproduction details; fix approval when the cause is only Likely or the fix is risky; unrelated changes in the diff; whether to run an independent review |
| [`database-change`](../skills/database-change/SKILL.md) | `database-specialist`; `code-reviewer` optional | **Always** plan approval; approval of each destructive step by name; which non-production environment may be used for validation; whether to run an independent review |
| [`pull-request-review`](../skills/pull-request-review/SKILL.md) | `code-reviewer` | Missing requirement; fetching from the remote; any follow-up such as drafting comments or fixing findings |

### How independence is preserved

When a workflow asks the `code-reviewer` for a review, it passes the **original requirement and the diff scope only**, never the implementer's report or reasoning. In `pull-request-review`, the coordinator also re-checks each Critical and High finding against the code before reporting, so you get fewer false alarms.

### Two-step delegation

The `new-feature` and `bug-fix` workflows first run their agent in **plan only** or **diagnose only** mode, so nothing changes before you have seen the plan. After approval, they resume the same agent, or start it again with the approved plan, to make the change.

## Procedure skills

| Skill | Preloaded into | Content |
|---|---|---|
| [`impact-analysis`](../skills/impact-analysis/SKILL.md) | All four agents | Dependencies, blast radius, regressions, tests, evidence levels, report template; [stack checklists](../skills/impact-analysis/stack-checklists.md) |
| [`feature-implementation`](../skills/feature-implementation/SKILL.md) | `feature-implementation` | Requirement, plan, checkpoint criteria, incremental implementation, tests, report |
| [`bug-fixing`](../skills/bug-fixing/SKILL.md) | `bug-investigation` | Reproduction, root cause, same-pattern search, minimal fix, regression test, report |
| [`database-analysis`](../skills/database-analysis/SKILL.md) | `database-specialist` | Schema mapping, integrity, locking, safe migration patterns, rollback, engine notes, query checklist, report |
| [`independent-review`](../skills/independent-review/SKILL.md) | `code-reviewer` | Review order, severity scale, verdict rules, report |

You can also use a procedure skill directly, without an agent and without a workflow. For example: `/personal-ai-workflow:impact-analysis what would renaming the Customer.Email column affect?`

## Limitations

- **Checkpoints are instructions.** A workflow tells Claude to stop and wait, and Claude normally does, but it's not enforced. Your permission settings still apply to every tool call.
- **Resuming an agent** depends on Claude Code being able to continue a finished subagent. If it can't, the workflow starts the agent again with the approved plan, which costs some repeated reading.
- **Agent names are namespaced** by the plugin name. If you install the agents in another way, such as by copying them into `~/.claude/agents/`, the workflows fall back to the names shown by `/agents`.
