---
name: bug-fix
description: End-to-end bug workflow with root-cause analysis, minimal fix, regression test, and approval checkpoints.
argument-hint: <bug description, error message, or failing test> [--full] [--review-model <model>]
disable-model-invocation: true
---

# Bug fix workflow

You coordinate this workflow from the main conversation. The user is here; the agents are not. You own the checkpoints (⏸), and the agents do the work.

**Bug report from the user:** $ARGUMENTS

## Coordinator rules

- At each ⏸, show the user what they need to decide and **wait for a clear answer**. Approval covers only what you showed.
- Agents don't see this conversation. Give each one everything it needs: symptom, steps to reproduce, environment, error messages (with secrets and personal data removed), and decisions so far.
- If an agent's report is unclear or its root cause is only Likely, say so plainly. Don't present a guess as the cause.
- Never commit, push, or open a pull request. The user decides that after the summary.

Agent names below assume the plugin is installed as `personal-ai-workflow`. If the agents were installed another way, use the names shown by `/agents`.

## Steps

1. **Gather the report.** If the report above is empty or lacks expected vs. actual behavior, ask for it. ⏸ Ask for missing reproduction details only if they block the investigation.
2. **Investigate and find the root cause.** Choose the path:
   - **Fast path**, unless the user wrote `--full`: when the report points to one clear, local area (an error with a stack trace, a failing test, one module), investigate it yourself, read-only, following the `bug-fixing` skill up to its proposed fix. Switch to the full path, passing on what you found, as soon as any of these is true: the root cause isn't clear after a focused look, the fix would touch more than about 3 files, or it involves a public contract, schema, security, CI/CD, infrastructure, or shared code.
   - **Full path** otherwise: delegate to `personal-ai-workflow:bug-investigation` with the full report and the instruction: **diagnose only: reproduce, find the root cause, analyze the impact, and propose the minimal fix; make no changes.**

   Tell the user which path you chose, in one line.
3. **Present the diagnosis:** root cause (Confirmed or Likely), how it was reproduced, the same pattern elsewhere, the impact, and the proposed fix.
4. **⏸ Approve the fix.** If the agent ended with **APPROVAL NEEDED**, the root cause is only Likely, or the blast radius is Medium or High, wait for explicit approval. Otherwise, say that you are proceeding and continue. Occurrences of the same pattern elsewhere are **not** fixed unless the user asks.
5. **Fix and add a regression test.** Fast path: do it yourself, following the `bug-fixing` skill. Full path: resume the same agent with "Approved: apply the fix and add the regression test", plus any changes the user asked for. If resuming isn't possible, start it again with the report and the approved fix. If the agent stops with **APPROVAL NEEDED** again, go back to step 4.
6. **Run tests.** Re-run the regression test and the related tests yourself. Confirm that the regression test exists and targets the bug. If it couldn't be written, make sure the reason is stated.
7. **Review the diff yourself.** Run `git status` and `git diff`. The change should be small and limited to the fix and its test. ⏸ Anything unrelated: show it and ask.
8. **Independent review, if warranted.** For blast radius Medium or High, or fixes touching security, data, or shared code, recommend a review by `personal-ai-workflow:code-reviewer`. ⏸ Ask, and if the user agrees, pass it the **original** bug report and the scope, not the fixer's reasoning. If the user wrote `--review-model <model>`, pass that model for this delegation.
9. **Summarize.**

## Summary format

```markdown
## Bug fixed: <title>
**Path:** fast or full, and why
**Root cause:** <cause>, Confirmed or Likely
**Fix:** <what changed>, files: <list>
**Regression test:** <name>, fails before and passes after: <yes/no/not possible, why>
**Tests:** run: <commands and results>; not run: <what and why>
**Same pattern elsewhere (not fixed):** <list or none found in scope>
**Review:** <verdict, or not requested>
**Remaining risks:** <list>
**Your next steps:** review the diff, then commit or open a pull request when ready
```
