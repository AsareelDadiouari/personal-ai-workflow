# Getting started

This guide takes you from installation to your first workflow in about ten minutes.

## Before you start: using it at work

If you plan to use this on an employer's or client's machine or code, first confirm that:

- the organization allows Claude Code, and the AI provider behind it, to process its code;
- installing third-party plugins is permitted (organizations can restrict plugin marketplaces through managed settings);
- your use follows its security, privacy, and AI usage policies.

The framework itself contains only generic instructions and sends nothing anywhere on its own. What leaves your machine is what Claude Code sends to its model provider during your session, under your organization's agreement with that provider.

## Requirements

- [Claude Code](https://code.claude.com/docs/en/overview) with plugin support (any recent version)
- Git
- A project to work in, ideally with a test suite

## Install

### Option A: from GitHub (recommended)

In Claude Code 2.1.275 or later:

```text
/plugin install personal-ai-workflow --marketplace AsareelDadiouari/personal-ai-workflow
```

On older versions, use two commands:

```text
/plugin marketplace add AsareelDadiouari/personal-ai-workflow
/plugin install personal-ai-workflow@personal-ai-workflow
```

The first command registers this repository as a plugin source. The second installs the plugin from it. To update later, run `/plugin marketplace update personal-ai-workflow`.

### Option B: from a local clone

Use this when you want to review the files before installing, or when GitHub access is restricted:

```bash
git clone https://github.com/AsareelDadiouari/personal-ai-workflow.git
```

Then, in Claude Code:

```text
/plugin marketplace add ./personal-ai-workflow
/plugin install personal-ai-workflow@personal-ai-workflow
```

### Option C: load for one session only

For trying it out or working on the framework itself:

```bash
claude --plugin-dir ./personal-ai-workflow
```

## Check the installation

In a Claude Code session:

1. Run `/agents`. You should see `personal-ai-workflow:feature-implementation`, `personal-ai-workflow:bug-investigation`, `personal-ai-workflow:database-specialist`, and `personal-ai-workflow:code-reviewer`.
2. Type `/personal-ai-workflow:` and look at the suggestions. You should see the four workflows and five procedure skills.

## Configure permissions (do this once)

The framework's rules are instructions. Real protection comes from Claude Code's permission rules. Add at least the deny and ask rules from [Recommended permission settings](security-principles.md#recommended-permission-settings) to `~/.claude/settings.json`, or to the project's `.claude/settings.json`. Adapt them to your environment.

## Give the project some context (optional, recommended)

Agents detect the stack on their own. A short `CLAUDE.md` **in the project** (not in this framework) makes them faster and more accurate. For example:

```markdown
# Project notes for Claude
- Build: `dotnet build`; tests: `dotnet test`; frontend tests: `npm test` in `web/`
- Migrations: EF Core, in `src/Data/Migrations`; never edit applied migrations
- Non-production database for validation: local Docker container `db-test`
- Tracker: Jira, project key SHOP; branch names like `feature/SHOP-123-short-title`
```

Keep project-specific and confidential details there, inside the project's own repository, never in this framework.

## Connect your ticket tracker (optional)

The `ticket` workflow reads tickets in one of three ways, in this order of preference:

1. **A tracker integration connected in Claude Code**, for example an Atlassian (Jira), Azure DevOps, or GitHub connector, if your organization provides one.
2. **The tracker's command-line tool**, signed in by you:

   | Tracker | Install and sign in |
   |---|---|
   | GitHub | Install [GitHub CLI](https://cli.github.com/), then run `gh auth login` |
   | Azure DevOps | Install the [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), then run `az extension add --name azure-devops`, `az login`, and optionally `az devops configure --defaults organization=https://dev.azure.com/<org> project=<project>` |
   | Jira | Install [jira-cli](https://github.com/ankitpokhrel/jira-cli) and run `jira init`, or use Atlassian's own CLI (`acli`) |

3. **Pasting the ticket text.** This needs no setup and works everywhere.

Sign-in always stays with you. The framework never asks for, types, or stores tokens.

**At work:** use only the tracker access your organization approves. If it doesn't allow CLI tokens or third-party connectors, paste the ticket instead. Ticket content is often confidential: it stays in the session and is never written to memory or to this framework.

Tell the workflow which tracker a project uses by adding a line to the project's `CLAUDE.md`, as in the example above. Then even a bare ticket number works.

## Your first runs

Start with something read-only to see how the agents report:

```text
/personal-ai-workflow:impact-analysis What would change if the order status field became an enum?
```

Then try a review of your current changes:

```text
/personal-ai-workflow:pull-request-review my uncommitted changes. Requirement: <what the change should do>
```

When you're comfortable, use a full workflow, either from a ticket or from a description:

```text
/personal-ai-workflow:ticket <ticket ID, URL, or pasted text>
/personal-ai-workflow:new-feature <requirement>
/personal-ai-workflow:bug-fix <bug report>
/personal-ai-workflow:database-change <schema change>
```

Each workflow stops at its checkpoints (⏸) and waits for your answer. Read the plan before approving it. The workflows are instructed never to commit or push, and your `ask` and `deny` permission rules back that up. When you're satisfied with the diff, commit it yourself.

## Using agents directly

You can also call an agent without a workflow:

```text
Use the personal-ai-workflow:code-reviewer agent to review the last commit against this requirement: ...
```

See [agents.md](agents.md) for what each agent does and when it stops for approval.

## Troubleshooting

| Problem | What to check |
|---|---|
| Agents or skills don't appear | Run `/plugin` and confirm `personal-ai-workflow` is installed and enabled. Restart the session after installing. |
| A workflow doesn't start | Workflows run only by explicit name, with the `personal-ai-workflow:` prefix. Claude won't start them on its own, by design. |
| An agent keeps asking for permission | Expected for writes and commands not covered by your `allow` rules. Allow read-only commands you trust, such as the project's test command, in your settings. |
| An agent stops with **APPROVAL NEEDED** | It needs your decision. Answer in the conversation, and Claude passes your answer back to the agent. |
| Results ignore your stack's conventions | Add the build and test commands and key conventions to the project's `CLAUDE.md`. |

## Uninstall

```text
/plugin uninstall personal-ai-workflow@personal-ai-workflow
/plugin marketplace remove personal-ai-workflow
```
