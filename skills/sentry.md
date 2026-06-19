---
description: Look up a Sentry issue by ID, or mark one as resolved
argument: "[resolve] <issue_id> - issue id alone fetches; prefix `resolve` to mark resolved"
---

Operate on Sentry issue `$ARGUMENTS`. Two subcommands:

- `<issue_id>` — fetch and display the issue (default)
- `resolve <issue_id>` — mark the issue as resolved

## Steps

1. Resolve the skill repo from the symlink:

   ```bash
   _SKILL_SRC=$(readlink ~/.claude/skills/sentry/SKILL.md)
   _REPO=$(dirname "$(dirname "$_SKILL_SRC")")
   ```

2. Dispatch on the first argument:

   - If `$ARGUMENTS` starts with the literal word `resolve`, run the resolve
     script with the issue id that follows:

     ```bash
     "$_REPO/bin/sentry-resolve" "<issue_id>"
     ```

   - Otherwise, treat `$ARGUMENTS` as an issue id and run the fetch script:

     ```bash
     "$_REPO/bin/sentry-fetch" "$ARGUMENTS"
     ```

3. Handle the script's exit code:

   - **0 (success)** —
     - For fetch: present the summary, then suggest what code might need
       fixing based on the stacktrace, referencing actual files in the codebase.
     - For resolve: confirm the issue is resolved and relay the status line
       printed by the script.

   - **3 (no config)** — ask the user: *"Should I create the stub at
     `.claude/sentry.json` (just this project) or `~/.claude/sentry.json`
     (global, shared across all projects)?"* Based on the answer, run one of:

     ```bash
     "$_REPO/bin/sentry-fetch" --init-project   # or --init-global
     ```

     The init command creates the file with `0600` permissions, adds it to
     `.gitignore` if project-level (when the cwd is a git repo), and prints
     where to edit it. Relay those instructions to the user. **Do not retry
     the fetch** — the user needs to edit the file with their real Sentry org
     slug and auth token first.

   - **4 (placeholder detected)** — the config exists but still contains
     placeholder values. Relay the script's message; the user needs to edit
     the file it points to.

   - **5 (forbidden — resolve only)** — the token is missing the
     `event:write` scope. Tell the user to edit the token at
     Sentry > Settings > Auth Tokens and add `event:write` (fetch only needs
     `event:read`).

   - **Other errors (exit 1)** — relay the script's error (network, API, JSON).
