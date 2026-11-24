# CLAUDE.md

Instructions for Claude Code when working **on this repository**, the Personal AI Workflow framework.

> Note: when this repository is installed as a plugin, this file is **not** loaded into other projects. Rules that agents and skills must follow in other projects are written into each agent and skill file directly. See [docs/architecture.md](docs/architecture.md#where-rules-live).

## What this repository is

A generic, reusable Claude Code plugin: subagents (`agents/`), skills (`skills/`, which hold both procedure skills and user-invoked workflow skills), and human documentation (`docs/`). It contains instructions and documentation only, not application code.

## Hard rules

1. **No client or employer material.** Never add proprietary source code, schemas, credentials, environment values, internal URLs, ticket text, business details, or names of clients or internal systems. Examples must be invented and generic (for example, `orders`, `customers`, `OrderService`).
2. **No secrets.** Not even fake-looking ones that could trigger scanners. Use obvious placeholders such as `<your-api-key>`.
3. **Use only real Claude Code formats.** Agent and skill frontmatter must use fields that Claude Code supports. If unsure whether a field or feature exists, check the official documentation or ask. Never invent configuration.
4. **No commits, pushes, or GitHub operations without the owner's explicit approval.**
5. **Stay in scope.** Change only the files the task needs. Do not reformat or restructure unrelated files.

## Writing conventions

- **Keep files short and direct.** Agents and skills are read by a model on every use, so every line costs context. Prefer checklists and concrete instructions over prose.
- **One responsibility per file.** An agent describes a role. A skill describes a procedure. A workflow describes the order and the human checkpoints. Avoid duplicating content: reference the skill instead of copying its steps.
- **Self-contained runtime rules.** Each agent and skill must state the safety rules it depends on, briefly, because it cannot rely on this file being loaded.
- **Stack-neutral by default.** Mention specific technologies (.NET, Angular, PostgreSQL, ...) only as "if the project uses X" guidance, never as an assumption.
- **Plain Markdown.** English, sentence-case headings, relative links.

## File formats

- **Agent** (`agents/<name>.md`): YAML frontmatter with `name`, `description` (when Claude should delegate to it), optional `tools` (comma-separated allowlist), `model`, `color`, and `skills` (YAML list of skills to preload; bare names work for skills in this plugin). The body is the system prompt. Plugin agents ignore `hooks`, `mcpServers`, and `permissionMode`; don't use them. Every agent needs **Non-negotiables**, **Approval protocol** (if it can write), and **Limitations** sections. Its procedure and report format live in the preloaded procedure skill, not in the agent file.
- **Skill** (`skills/<name>/SKILL.md`): YAML frontmatter with `name` and `description` (what it does and when to use it). Workflow skills also set `disable-model-invocation: true` and an `argument-hint`, and read the user's input from `$ARGUMENTS`. The body is the procedure. Supporting files may sit next to `SKILL.md`.
- **Skill names must not match Claude Code's built-in skills** (for example, `code-review`, `security-review`, `simplify`, `init`). A bare name in an agent's `skills:` list resolves to the built-in skill first. This happened once and is why the review procedure is named `independent-review`.
- **Plugin manifest** (`.claude-plugin/plugin.json`): name, version, description.

## Before considering a change done

- Re-read the diff (`git diff`).
- Check that links resolve and that the README component tables still match the files.
- Check consistency: agents, skills, and workflows must not contradict each other or `docs/security-principles.md`.
- Search for anything resembling a secret, a real company name, or a real internal URL.
