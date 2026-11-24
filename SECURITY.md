# Security policy

## Scope

This repository contains Markdown instructions and configuration for Claude Code. It contains no application code or services. Security issues here are most likely to be one of these:

- An instruction that could lead an agent to perform a destructive or unauthorized action.
- An instruction that weakens the [security principles](docs/security-principles.md) or the client data protection policy.
- Accidentally committed sensitive information (credentials, personal data, or third-party proprietary material).

## Reporting a problem

Please **do not open a public issue** for anything involving sensitive information.

Use GitHub's private vulnerability reporting: **Security** tab → **Report a vulnerability**. For non-sensitive problems with an instruction, a regular issue is fine.

Include:

- the file and section concerned,
- what could go wrong and under what conditions,
- a suggested fix, if you have one.

## If sensitive data was committed

Removing it in a new commit is **not enough**, because it remains in Git history. Revoke or rotate any exposed credential first, then clean the history and coordinate with the owner.

## Using this framework safely

This framework guides model behavior. It is not a security control. Read [Prompts are not security boundaries](docs/security-principles.md#prompts-are-not-security-boundaries) and configure real permissions before using it on any codebase you care about.
