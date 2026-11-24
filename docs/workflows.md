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
| [`new-feature`](../skills/new-feature/SKILL.md) | Full path: `feature-implementation`, then `code-reviewer`. Fast path: `code-reviewer` only | Ambiguous requirements; plan approval when the change is substantial or risky; failing tests; unexpected files in the diff; which review findings to fix |
| [`bug-fix`](../skills/bug-fix/SKILL.md) | Full path: `bug-investigation`. Either path: `code-reviewer` when risky | Missing reproduction details; fix approval when the cause is only Likely or the fix is risky; unrelated changes in the diff; whether to run an independent review |
| [`database-change`](../skills/database-change/SKILL.md) | `database-specialist` always (no fast path); `code-reviewer` optional | **Always** plan approval; approval of each destructive step by name; which non-production environment may be used for validation; whether to run an independent review |
| [`pull-request-review`](../skills/pull-request-review/SKILL.md) | Full path: `code-reviewer`. Fast path: none | Missing requirement; fetching from the remote; any follow-up such as drafting comments or fixing findings |

### Fast path and full path

Starting an agent costs tokens: it begins with an empty context, loads its instructions, and reads the code again. For small changes, that cost buys little. So `new-feature`, `bug-fix`, and `pull-request-review` choose a path first and tell you which one they picked:

| Workflow | Fast path when | What the fast path skips |
|---|---|---|
| `new-feature` | About 3 files or fewer; no public API, contract, schema, security, authentication, CI/CD, or infrastructure change; no new dependency; no important Unknowns | The implementer agent. The main conversation implements it. **The independent review still runs.** |
| `bug-fix` | The report points to one clear, local area and the root cause is found quickly | The investigation agent. The workflow switches to the full path as soon as the bug turns out to be broader. |
| `pull-request-review` | About 150 changed lines and 3 files or fewer, and the main conversation has not written or discussed the change | The reviewer agent. The main conversation follows the same review procedure. |

`database-change` has no fast path: database changes always go through the specialist.

Options you can add to the command:

- `--full`: always use the full path.
- `--review-model <model>`: run the independent reviewer on another model, for example `--review-model sonnet`. This applies to the reviewer agent only, and it trades some review depth for lower cost. To change the model of **all** subagents instead, Claude Code supports the `CLAUDE_CODE_SUBAGENT_MODEL` environment variable together with `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` (see [model selection for subagents](https://code.claude.com/docs/en/sub-agents)).

Example:

```text
/personal-ai-workflow:new-feature Add a "copy link" button to the order details page --review-model sonnet
```

### How independence is preserved

When a workflow asks the `code-reviewer` for a review, it passes the **original requirement and the diff scope only**, never the implementer's report or reasoning. In `pull-request-review`, the coordinator also re-checks each Critical and High finding against the code before reporting, so you get fewer false alarms.

### Two-step delegation

On the full path, the `new-feature` and `bug-fix` workflows first run their agent in **plan only** or **diagnose only** mode, so nothing changes before you have seen the plan. After approval, they resume the same agent, or start it again with the approved plan, to make the change.

## Procedure skills

| Skill | Preloaded into | Content |
|---|---|---|
| [`impact-analysis`](../skills/impact-analysis/SKILL.md) | `feature-implementation`, `bug-investigation`, `database-specialist` | Dependencies, blast radius, regressions, tests, evidence levels, report template; [stack checklists](../skills/impact-analysis/stack-checklists.md) |
| [`feature-implementation`](../skills/feature-implementation/SKILL.md) | `feature-implementation` | Requirement, plan, checkpoint criteria, incremental implementation, tests, report |
| [`bug-fixing`](../skills/bug-fixing/SKILL.md) | `bug-investigation` | Reproduction, root cause, same-pattern search, minimal fix, regression test, report |
| [`database-analysis`](../skills/database-analysis/SKILL.md) | `database-specialist` | Schema mapping, integrity, locking, safe migration patterns, rollback, engine notes, query checklist, report |
| [`independent-review`](../skills/independent-review/SKILL.md) | `code-reviewer` | Review order, evidence labels and blast-radius scale (a compact version of impact analysis), severity scale, verdict rules, report |

You can also use a procedure skill directly, without an agent and without a workflow. For example: `/personal-ai-workflow:impact-analysis what would renaming the Customer.Email column affect?`

## Limitations

- **Checkpoints are instructions.** A workflow tells Claude to stop and wait, and Claude normally does, but it's not enforced. Your permission settings still apply to every tool call.
- **Resuming an agent** depends on Claude Code being able to continue a finished subagent. If it can't, the workflow starts the agent again with the approved plan, which costs some repeated reading.
- **Agent names are namespaced** by the plugin name. If you install the agents in another way, such as by copying them into `~/.claude/agents/`, the workflows fall back to the names shown by `/agents`.
