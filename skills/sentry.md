---
description: Look up a Sentry issue by ID and display error details with stacktrace
argument: issue_id - The Sentry issue ID (numeric) or short ID (e.g. DRIVETIME-API-3A)
---

Look up Sentry issue `$ARGUMENTS` and display the error details.

## Steps

1. Run the fetch script. It resolves this repo via the skill's symlink, reads
   `.claude/sentry.json` (project-level, preferred) or `~/.claude/sentry.json`
   (global fallback), fetches the issue, and prints a formatted summary.

   ```bash
   _SKILL_SRC=$(readlink ~/.claude/skills/sentry/SKILL.md)
   _REPO=$(dirname "$(dirname "$_SKILL_SRC")")
   "$_REPO/bin/sentry-fetch" "$ARGUMENTS"
   ```

2. Handle the script's exit code:

   - **0 (success)** — present the summary, then suggest what code might need
     fixing based on the stacktrace, referencing actual files in the codebase.

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

   - **Other errors (exit 1)** — relay the script's error (network, API, JSON).
