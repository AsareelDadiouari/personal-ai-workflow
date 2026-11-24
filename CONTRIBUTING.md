# Contributing

This is a personal framework, but suggestions and fixes are welcome.

## Ground rules

1. **No proprietary or client material.** Do not contribute code, schemas, documentation, or examples taken from an employer or client, even anonymized. Use invented examples. See the [client data protection policy](docs/security-principles.md#client-data-protection-policy).
2. **No secrets**, real or realistic-looking.
3. **Use only real Claude Code formats.** Agent and skill frontmatter must use supported fields. Link to the official documentation when introducing a feature that is new to the repository.
4. **Keep it small.** Prefer improving an existing agent or skill over adding a new one. Each file should have one responsibility.
5. **Stay consistent.** Agents, skills, and workflows must not contradict each other or [docs/security-principles.md](docs/security-principles.md).

## Making a change

1. Open an issue first for new agents, skills, or workflows, to discuss scope.
2. Keep pull requests focused on one change.
3. Describe what changed, why, and how you checked it (for example, by running the skill in a sample project).
4. Update the README component tables and any affected documentation.

## Style

- English, plain Markdown, relative links.
- Short, direct instructions. Checklists over prose.
- Technology-specific advice goes in conditional sections ("If the project uses X…").
