---
name: new-feature
description: End-to-end workflow for a new feature. Covers requirements, impact analysis, plan, approval, implementation, tests, diff review, independent code review, and summary. Delegates to the feature-implementation and code-reviewer agents.
argument-hint: <feature description or requirement>
disable-model-invocation: true
---

# New feature workflow

You coordinate this workflow from the main conversation. The user is here; the agents are not. You own the checkpoints (⏸), and the agents do the work.

**Requirement from the user:** $ARGUMENTS

## Coordinator rules

- At each ⏸, show the user what they need to decide and **wait for a clear answer**. Approval covers only what you showed.
- Agents don't see this conversation. Give each one everything it needs in its task: the requirement, acceptance criteria, constraints, and decisions so far.
- If an agent's report is unclear, incomplete, or contradicts itself, tell the user instead of filling the gaps yourself.
- Never commit, push, or open a pull request. The user decides that after the summary.

Agent names below assume the plugin is installed as `personal-ai-workflow`. If the agents were installed another way, use the names shown by `/agents`.

## Steps

1. **Understand the requirement.** If the requirement above is empty, ask for it. Restate it and list acceptance criteria and open questions. ⏸ If anything is ambiguous, ask before continuing.
2. **Impact analysis and plan.** Delegate to `personal-ai-workflow:feature-implementation` with the requirement, acceptance criteria, constraints, and the instruction: **plan only: run the impact analysis and produce the plan; make no changes.**
3. **Present the plan.** Show the change points, blast radius, risks, Unknowns, and the test plan. Keep it short, and link to `file:line` where useful.
4. **⏸ Approval.** If the agent ended with **APPROVAL NEEDED**, or the blast radius is Medium or High, wait for explicit approval, including any changes the user asks for. If the blast radius is Low and nothing was flagged, say that you are proceeding and continue.
5. **Implement.** Resume the same agent with "Approved: implement the plan", plus any changes the user asked for. If resuming isn't possible, start the agent again with the requirement and the approved plan. If the agent stops with **APPROVAL NEEDED** again, go back to step 4.
6. **Run tests.** Re-run the test commands the agent reported to confirm the results yourself. If tests fail, show the failures. ⏸ Ask whether to fix them before review.
7. **Review the diff yourself.** Run `git status` and `git diff`. Every changed file must be part of the approved plan. ⏸ Unexpected files or changes: show them and ask.
8. **Independent review.** Delegate to `personal-ai-workflow:code-reviewer` with the **original** requirement and acceptance criteria, and the scope to review (for example, uncommitted changes). **Do not pass the implementer's report or reasoning**, so the review stays independent.
   - If the verdict is **Request changes**: show the findings. ⏸ Ask which findings to fix, send those to the implementer, then review again.
   - After two review rounds, stop and hand the remaining findings to the user.
9. **Summarize.**

## Summary format

```markdown
## Feature delivered: <title>
**What changed:** <behavior now available>
**Files:** <list with one-line purpose>
**Tests:** run: <commands and results>; not run: <what and why>
**Review:** <verdict>; open findings: <list or none>
**Remaining risks and unknowns:** <list>
**Your next steps:** review the diff, then commit or open a pull request when ready
```
