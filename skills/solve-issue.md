---
description: Solve GitHub issues by reading them from the repo and using their content as a prompt
argument: "[repo] [issue_number|all]" — repo is owner/repo (optional, defaults to current repo); issue_number picks one issue; "all" solves every open issue; omit both to list open issues
---

# Solve GitHub Issue

A skill that reads GitHub issues from a GitHub repo and uses them as the prompt to
implement a fix or feature.

## Prerequisites

- The `gh` CLI must be installed and authenticated (`gh auth status`)
- **GITHUB_TOKEN conflict:** If `gh` commands fail with 401/bad credentials, an invalid
  `GITHUB_TOKEN` env var is likely overriding `gh`'s stored credentials. Automatically
  prefix `gh` commands with `GITHUB_TOKEN= ` to clear it. Do not prompt the user about this.

## Workflow

### Step 0: Parse arguments

`$ARGUMENTS` can be any of the following forms:

| Example input | Repo | Issue |
|---|---|---|
| *(empty)* | current repo | list & prompt |
| `42` | current repo | #42 |
| `all` | current repo | all open issues |
| `owner/repo` | owner/repo | list & prompt |
| `owner/repo 42` | owner/repo | #42 |
| `owner/repo all` | owner/repo | all open issues |

**Parsing rules:**
1. Split `$ARGUMENTS` on whitespace into tokens.
2. If a token contains `/` (e.g. `spinrag/drivetime`), treat it as the **repo**.
3. If a token is a number, treat it as the **issue number**.
4. If a token is the word `all` (case-insensitive), set mode to **all**.
5. Anything left over is an error — inform the user of the expected format.

**Determining the repo when none was provided:**
- If the working directory is a git repo with a GitHub remote, detect the repo from `gh repo view --json nameWithOwner -q .nameWithOwner`.
- If there is **no GitHub remote**, ask the user: **"Which repo would you like to work on? (format: owner/repo)"** and wait for a response.

For all `gh` commands targeting a non-current repo, pass `--repo <owner/repo>` (abbreviated `-R`).

### Step 1: Identify the issue(s)

**Mode: single issue** (issue number was provided):

```bash
gh issue view <number> -R <repo> --json number,title,body,labels,assignees,milestone,state,comments
```

- If the issue is **closed**, note it but proceed anyway (the user explicitly asked for it).
- If the issue **does not exist**, report the error and stop.

**Mode: all** (`all` was provided):

```bash
gh issue list -R <repo> --state open --limit 30 --json number,title,labels,assignees,body
```

- Print a brief list of the open issues (number + title) so the user can follow along.
- Proceed through each issue sequentially, following Steps 2–5 for each. Do NOT pause between issues.
- Between issues, print a one-line summary of what was done and move on.
- If an issue is unclear or blocked, skip it with a note and move to the next.

**Mode: list & prompt** (no issue number, no `all`):

```bash
gh issue list -R <repo> --state open --limit 30 --json number,title,labels,assignees
```

- Present the open issues as a **numbered list** with title, labels, and assignee.
- Ask the user: **"Which issue would you like to work on? (enter a number, or 'all')"**
- Wait for the user to choose before proceeding. This is the **only** prompt in the workflow.

### Step 2: Understand the issue

Read the full issue body and comments. Evaluate whether the issue contains enough
information to act on.

**If the issue is actionable** (has a bug description, feature request, or clear task):
proceed directly to Step 3. Do not ask for confirmation.

**If the issue is truly unworkable** (empty body AND vague title with no comments giving
context): only then ask the user for clarification. In `all` mode, skip the issue with a
note instead of prompting.

When in doubt, use your best judgment and proceed. State any assumptions inline as you
implement — the user can course-correct in review.

### Step 3: Plan and implement

1. **Explore the codebase** — Use Grep, Glob, and Read to understand the relevant files,
   patterns, and conventions already in use.
2. **State your plan briefly** — A few bullet points: which files, what approach. Then
   immediately begin implementing. Do NOT ask for confirmation.
3. **Implement** — Follow existing code conventions, keep changes minimal and focused.
   Include or update tests if the project has a test suite.

### Step 4: Verify and report

After implementation:

1. **Run relevant checks** (detect automatically: `pnpm test`, `pnpm typecheck`, etc.)
2. **Summarize** — files changed, how it works, any follow-ups.
3. **Suggest a commit message** following conventional commits:
   ```
   fix(scope): short description (#<issue-number>)

   <body explaining what and why>

   Closes #<issue-number>
   ```

In `all` mode, run checks once after all issues are done rather than after each one.

## Error Handling

- If `gh` is not installed: tell the user to install it (`brew install gh` / `sudo apt install gh`)
- If not authenticated: tell the user to run `gh auth login`
- If no GitHub remote and no repo argument: prompt the user for `owner/repo`
- If the provided repo doesn't exist or the user lacks access: report the error and stop
- If the API rate-limits: wait and retry, or inform the user

## Guidelines

- **Minimize prompts.** The only time you should stop and ask the user a question is during
  "list & prompt" mode (no issue specified) or when an issue is truly unworkable. Everything
  else — exploring, planning, implementing, verifying — should flow without interruption.
- Always read the issue AND its comments — context is often in the thread
- Respect labels: if labeled `bug`, treat it as a bug fix; if `enhancement`, treat as a feature
- If the issue references other issues or PRs, read those too for context
- Never push code or create PRs automatically — leave that to the user
- If the issue is too large for a single implementation pass, break it into steps and
  implement phase by phase without asking permission between phases
