---
name: new-feature
description: End-to-end feature workflow with impact analysis, approval checkpoints, tests, and independent review.
argument-hint: <requirement> [--full] [--review-model <model>]
disable-model-invocation: true
---

# New feature workflow

You coordinate this workflow from the main conversation. The user is here; the agents are not. You own the checkpoints (⏸).

**Requirement from the user:** $ARGUMENTS

## Coordinator rules

- At each ⏸, show the user what they need to decide and **wait for a clear answer**. Approval covers only what you showed.
- Agents don't see this conversation. Give each one everything it needs in its task: the requirement, acceptance criteria, constraints, and decisions so far.
- If an agent's report is unclear, incomplete, or contradicts itself, tell the user instead of filling the gaps yourself.
- Never commit, push, or open a pull request. The user decides that after the summary.

Agent names below assume the plugin is installed as `personal-ai-workflow`. If the agents were installed another way, use the names shown by `/agents`.

## Steps

1. **Understand the requirement.** If the requirement above is empty, ask for it. Restate it and list acceptance criteria and open questions. ⏸ If anything is ambiguous, ask before continuing.
2. **Choose the path.** Do a quick, read-only scoping pass yourself: locate the change points and their direct callers.
   - **Fast path** if **all** of these hold, and the user did not write `--full`:
     - about 3 files or fewer,
     - no public API, contract, or schema change,
     - no security, authentication, CI/CD, or infrastructure code,
     - no new dependency,
     - no important Unknowns.
   - **Full path** otherwise. Tell the user which path you chose, in one line, and why.

### Fast path (small, low-risk changes)

3. **Implement it yourself** by following the `feature-implementation` skill, including its `impact-analysis` step, scaled to the size of the change. If at any point the criteria above stop holding, stop and switch to the full path, passing on what you found.
4. **Run tests** and **review the diff yourself** (`git status`, `git diff`). ⏸ Show any unexpected files or changes, and ask.
5. **Independent review** by `personal-ai-workflow:code-reviewer`: same as full-path step 9.
6. **Summarize.**

### Full path

3. **Impact analysis and plan.** Delegate to `personal-ai-workflow:feature-implementation` with the requirement, acceptance criteria, constraints, your scoping notes, and the instruction: **plan only: run the impact analysis and produce the plan; make no changes.**
4. **Present the plan.** Show the change points, blast radius, risks, Unknowns, and the test plan. Keep it short, with `file:line` references where useful.
5. **⏸ Approval.** If the agent ended with **APPROVAL NEEDED**, or the blast radius is Medium or High, wait for explicit approval, including any changes the user asks for. Otherwise, say that you are proceeding and continue.
6. **Implement.** Resume the same agent with "Approved: implement the plan", plus any changes the user asked for. If resuming isn't possible, start it again with the requirement and the approved plan. If it stops with **APPROVAL NEEDED** again, go back to step 5.
7. **Run tests.** Re-run the test commands the agent reported to confirm the results yourself. ⏸ If tests fail, show the failures and ask whether to fix them before review.
8. **Review the diff yourself.** Run `git status` and `git diff`. Every changed file must be part of the approved plan. ⏸ Unexpected files or changes: show them and ask.
9. **Independent review.** Delegate to `personal-ai-workflow:code-reviewer` with the **original** requirement and acceptance criteria, and the scope to review (for example, uncommitted changes). **Do not pass the implementer's report or reasoning.** If the user wrote `--review-model <model>`, pass that model for this delegation.
   - If the verdict is **Request changes**: show the findings. ⏸ Ask which ones to fix, fix them (fast path: yourself; full path: through the implementer), then review again.
   - After two review rounds, stop and hand the remaining findings to the user.
10. **Summarize.**

## Summary format

```markdown
## Feature delivered: <title>
**Path:** fast or full, and why
**What changed:** <behavior now available>
**Files:** <list with one-line purpose>
**Tests:** run: <commands and results>; not run: <what and why>
**Review:** <verdict>; open findings: <list or none>
**Remaining risks and unknowns:** <list>
**Your next steps:** review the diff, then commit or open a pull request when ready
```
