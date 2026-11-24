# Architecture

This document explains how the framework is organized and why.

## Target environment

The framework targets **Claude Code** and uses only mechanisms Claude Code supports natively:

| Concept | Claude Code mechanism | Location |
|---|---|---|
| Packaging | Plugin (`.claude-plugin/plugin.json`) | repository root |
| Specialized agents | Subagents: Markdown with YAML frontmatter (`name`, `description`, optional `tools`, `model`, `color`, `skills`) | `agents/*.md` |
| Procedures and workflows | Agent Skills: `SKILL.md` with YAML frontmatter (`name`, `description`; workflows add `disable-model-invocation`, `argument-hint`) | `skills/<name>/SKILL.md` |
| Enforcement | Permission rules in settings files (configured by the user, not shipped) | `~/.claude/settings.json` or project `.claude/settings.json` |

Other AI coding tools are not supported directly. The Markdown bodies are mostly tool-neutral and could be adapted later, but frontmatter and loading behavior are Claude Code-specific.

## Why a plugin

- **No copying into projects.** Installing the plugin makes the agents and skills available in every project. Nothing from the framework lands in the project, and nothing from the project lands in the framework. This supports the [client data protection policy](security-principles.md#client-data-protection-policy).
- **Versioned and updatable.** One repository, one version number, updates via Git.
- **Namespaced.** You invoke plugin skills with the plugin prefix (for example, `/personal-ai-workflow:impact-analysis`), so slash commands don't collide with a project's own skills. **Exception:** bare names in an agent's `skills:` preload list resolve to Claude Code's built-in skills first. That's why no skill here shares a name with a built-in one, and why the review procedure is `independent-review` rather than `code-review`.

**Alternative:** copying `agents/` and `skills/` into `~/.claude/agents/` and `~/.claude/skills/` also works and gives shorter names. See the limitations in [workflows.md](workflows.md#limitations).

## Responsibilities

```
          Workflow (order + human checkpoints)
               │
      ┌────────┼──────────────┐
      ▼        ▼              ▼
   Skills   Agents        Human approval
 (how to do (who does it,
  a task)    with which tools)
```

- **Skills** describe *how* to perform one procedure (impact analysis, root-cause investigation, review). They are reusable by any agent or by the main conversation.
- **Agents** describe *who*: a role with a focused system prompt and a restricted tool list. Agents preload skills; they don't duplicate their steps. Only the core non-negotiables are repeated in both places on purpose, so each file stays safe on its own.
- **Workflows** describe *in what order*, and *where a human must approve* before continuing.

**Impact analysis** is the central skill. The implementation, bug, and database agents must run it before changing anything. The reviewer uses it to judge blast radius.

## Where rules live

Claude Code does not load a plugin's `CLAUDE.md` or `docs/` into the user's projects. Therefore:

- `docs/security-principles.md` is the **reference** for people. It is the source of truth.
- Every agent and skill carries a short **"Non-negotiables"** section that restates the rules it depends on. Keeping these short and consistent with the reference is part of every review.
- The repository's own `CLAUDE.md` only governs work **on** this repository.

## Tool restrictions

Tool lists are a real control, unlike prompt text, so each agent gets the narrowest list that lets it do its job:

| Agent | Tools | Rationale |
|---|---|---|
| `code-reviewer` | Read, Grep, Glob, Bash | Must not edit files. Bash is needed for `git diff` / `git log` and running tests. Bash could still modify files, so this is a reduction, not a guarantee. |
| `database-specialist` | Read, Grep, Glob, Bash, Edit, Write | Analyzes schema and migration files and drafts migrations after approval. It has no database connection unless the user configures and authorizes one. |
| `bug-investigation` | Read, Grep, Glob, Bash, Edit, Write | Needs to reproduce, fix, and add regression tests. |
| `feature-implementation` | Read, Grep, Glob, Bash, Edit, Write | Implements and tests. |

No agent has the `Agent` tool, so none can spawn further subagents.

## Skill preloading and approvals

- Each agent lists `impact-analysis` and its own procedure skill in its `skills` frontmatter. Claude Code injects the full skills into the agent's context at startup. This was tested with the plugin loaded through `--plugin-dir`: the bare skill name works for a skill in the same plugin.
- Subagents cannot talk to the user. When approval is needed, an agent stops and returns a section titled **APPROVAL NEEDED**. The main conversation shows it to the user and resumes the agent with the decision. See [agents.md](agents.md#approval-protocol).
- Workflow skills use `disable-model-invocation: true` so they run only when the user calls them. Skills with that setting cannot be preloaded into agents, which fits the split: agents preload procedure skills only.
- Plugin agents ignore the `hooks`, `mcpServers`, and `permissionMode` frontmatter fields, so the framework does not use them.

## Workflows

**Decided: workflows are user-invocable skills.** Claude Code has no dedicated "workflow" file format, so each workflow is a skill you start by name, for example `/personal-ai-workflow:new-feature`. It walks through the steps in order, delegates to the right agents, and stops at the human checkpoints. There is no separate `workflows/` folder, so each workflow's steps exist in exactly one place.

The alternative, workflows as documentation only, was rejected: Claude would only follow them when explicitly asked to read them.

Skills therefore come in two kinds:

| Kind | Skills | Started by | Content |
|---|---|---|---|
| **Workflow skills** | `new-feature`, `bug-fix`, `database-change`, `pull-request-review` | The user, by name | Order of steps, which agent handles each step, approval checkpoints. They reference procedure skills instead of repeating their steps. |
| **Procedure skills** | `impact-analysis`, `feature-implementation`, `bug-fixing`, `database-analysis`, `independent-review` | Agents, the main conversation, or Claude automatically when relevant | How to perform one procedure well. |

Workflow skills should run only when the user asks for them, never because Claude decided a workflow fits. The supported frontmatter `disable-model-invocation: true` enforces this.

Agents run in their own context, so a workflow passes each agent the information it needs, such as the requirement and the impact analysis, and the agent reports back. The approval checkpoints are handled in the main conversation, where the user is present.

## Technology adaptation

Agents start by detecting the stack from evidence: project files (`*.csproj`, `package.json`, `angular.json`, `Dockerfile`, migration folders, CI files), and the project's own `CLAUDE.md`. Technology-specific guidance is written as conditional sections ("If the project uses EF Core migrations…"), never as defaults.

## Evolution

Keep the initial version small. Add a component only when a repeated need appears. Candidates for later:

- Optional hooks that block clearly dangerous commands (a real control, but platform-specific).
- Additional skills (for example, a dependency upgrade or a performance investigation).
