# Tracker reference

Read only the section for the tracker in use. CLI commands and flags change between versions: if one fails, check `--help` instead of guessing.

## Recognizing a reference

| Tracker | URL patterns | Short forms |
|---|---|---|
| **Azure DevOps** | `https://dev.azure.com/<org>/<project>/_workitems/edit/<id>`, `https://<org>.visualstudio.com/<project>/_workitems/edit/<id>` | `AB#<id>` (in GitHub-linked repos); a bare number if the project's `CLAUDE.md` says the tracker is Azure DevOps |
| **GitHub** | `https://github.com/<owner>/<repo>/issues/<n>` (or `/pull/<n>`) | `<owner>/<repo>#<n>`; `#<n>` for the current repository's remote |
| **Jira** | `https://<site>.atlassian.net/browse/<KEY-123>`, or a self-hosted `/browse/<KEY-123>` | `<KEY>-<number>`, for example `SHOP-482` |

A bare number with no configured tracker is ambiguous: ask.

## Fetching a ticket

Prefer, in order:

1. **A connected integration** in the session, if one exists for this tracker. Look for available tools whose names mention the tracker (for example Atlassian or Jira, Azure DevOps, or GitHub), and use their read operations only.
2. **The tracker's CLI**, if it is installed and authenticated (commands below).
3. **Pasted text.** Ask the user to paste the title, type, description, acceptance criteria or reproduction steps, and relevant comments. This always works, including under strict company policies.

Never call a tracker's REST API with a token yourself, and never ask for one.

### GitHub (`gh`)

```bash
gh issue view <n> --repo <owner>/<repo> --json number,title,body,labels,state,url,comments
```

Use `gh pr view` for a pull request. Not authenticated: the user runs `gh auth login`.

### Azure DevOps (`az` with the `azure-devops` extension)

```bash
az boards work-item show --id <id> --org https://dev.azure.com/<org> --output json
```

Not set up: the user runs `az extension add --name azure-devops`, then `az login`, and optionally `az devops configure --defaults organization=https://dev.azure.com/<org> project=<project>`. Description fields are HTML: convert them to plain text. Images are not included; ask before downloading any attachment.

### Jira

- **jira-cli:** `jira issue view <KEY-123> --plain`, or `--raw` for JSON. Setup: the user runs `jira init`.
- **Atlassian CLI (`acli`):** check `acli jira --help` for the work-item view command.

Jira Cloud descriptions may arrive as structured JSON (Atlassian Document Format): extract the text.

## Type fields

| Tracker | Where the type is | Usual values → class |
|---|---|---|
| **Azure DevOps** | `fields."System.WorkItemType"`; repro steps in `Microsoft.VSTS.TCM.ReproSteps`; acceptance criteria in `Microsoft.VSTS.Common.AcceptanceCriteria`; tags in `System.Tags` | Bug → Bug. User Story, Product Backlog Item, Feature, Requirement → Feature. Task → decide by content (check the parent item). Epic → too big: suggest splitting. |
| **GitHub** | Issue type (if the organization uses them) and labels | `bug`, `defect`, `regression` → Bug. `enhancement`, `feature` → Feature. `chore`, `refactor`, `documentation`, `question` → Other. No label → decide by content. |
| **Jira** | `fields.issuetype.name`; labels; acceptance criteria often live in a custom field or the description | Bug, Defect, Incident → Bug. Story, New Feature, Improvement → Feature. Task, Sub-task → decide by content (check the parent). Epic → suggest splitting. Spike → Other. |

Teams customize types. When a value isn't listed, decide by content and say so.

## Linking conventions

Suggest these for the user's branch and commit. The workflows never commit themselves.

| Tracker | Branch example | Commit or PR reference |
|---|---|---|
| **Azure DevOps** | `bugfix/1234-short-title` or `feature/1234-short-title` | `AB#1234` in GitHub repos; `#1234` or a linked work item in Azure Repos |
| **GitHub** | `fix/123-short-title` or `feat/123-short-title` | `Fixes #123` or `Closes #123` in the PR description, which closes the issue on merge |
| **Jira** | `bugfix/SHOP-482-short-title` | The issue key (`SHOP-482`) in the branch name, commit message, and PR title |

If the project's `CLAUDE.md` or contributing guide defines its own conventions, use those instead.
