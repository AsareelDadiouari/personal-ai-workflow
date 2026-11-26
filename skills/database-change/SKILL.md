---
name: database-change
description: End-to-end schema change or migration workflow with rollback plan, mandatory approval, and non-production validation.
argument-hint: <schema change or migration needed>
disable-model-invocation: true
---

# Database change workflow

You coordinate this workflow from the main conversation. The user is here; the agents are not. You own the checkpoints (⏸), and the agents do the work.

**Requested change from the user:** $ARGUMENTS

## Coordinator rules

- At each ⏸, show the user what they need to decide and **wait for a clear answer**. Approval covers only what you showed.
- **Never production.** No step of this workflow connects to, or runs anything against, a production database. If an environment's status is unclear, treat it as production.
- **Never pass credentials or connection strings** into an agent's task. If the agent needs a database, the user configures access in the environment, and the agent refers to it only by name.
- Never commit, push, or open a pull request. The user decides that after the summary.

Agent names below assume the plugin is installed as `personal-ai-workflow`. If the agents were installed another way, use the names shown by `/agents`.

## Steps

1. **Understand the change.** If the request above is empty, ask for it. If it is a ticket ID or URL, follow the `ticket-intake` skill first and use its extracted requirement. Restate it, including why it is needed and which application changes depend on it.
2. **Inspect, analyze, and propose.** Delegate to `personal-ai-workflow:database-specialist` with the request and context. The agent detects the engine and migration tooling, maps the schema, runs the impact and integrity analysis, and proposes the migration and rollback plan. Its procedure always stops for approval before drafting.
3. **Present the proposal:** integrity risks, locking and duration (with Assumed values labeled), deployment ordering, rollback, irreversible steps, and Unknowns.
4. **⏸ Approve the plan.** Always wait for explicit approval. Call out any destructive or irreversible step (drop, truncate, broad delete or update, type narrowing, column removal) separately, and require approval for each one by name.
5. **Draft the migration.** Resume the same agent with "Approved: draft the migration only; do not apply or validate it", plus any changes the user asked for. If resuming isn't possible, start it again with the request and the approved plan. If the agent stops with **APPROVAL NEEDED**, go back to step 4.
6. **Review the files yourself.** Run `git status` and `git diff`. Only the files named in the approved plan should have changed: migration and SQL files and, for ORM-based migrations, the listed model or mapping files and generated artifacts (such as a model snapshot). Check that the generated SQL matches the approved plan. ⏸ Anything else: show it and ask.
7. **⏸ Validate in an authorized non-production environment.** Ask the user which environment, if any, is authorized for validation (for example, a local container or a dedicated test database). If one is authorized, resume the agent with "Approved: apply and roll back the migration on `<environment name>`, which the user has authorized for this task; run nothing else that writes", and report the results. If the agent stops with **APPROVAL NEEDED**, show it and ask again. If no environment is authorized, record the migration as **not validated**.
8. **Independent review, recommended.** ⏸ Offer a review by `personal-ai-workflow:code-reviewer`, passing the original request and the migration files.
9. **Document the results.** Produce the summary. Any application code changes that depend on the schema change belong in a separate `new-feature` or `bug-fix` workflow.

## Summary format

```markdown
## Database change: <title>
**Change:** <what and why>
**Migration files:** <list>
**Rollback:** <how; data loss: yes/no>
**Validation:** <environment, apply and rollback results>, or not validated: <why>
**Deployment notes:** <ordering with application releases, backfills, expected locking>
**Review:** <verdict, or not requested>
**Unknowns and remaining risks:** <list>
**Your next steps:** review the migration, then commit or open a pull request when ready; schedule the deployment with whoever owns the database
```
