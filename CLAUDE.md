# gstack-extend-cto

Personal extension skills. Versioning source of truth: `VERSION` file.

## Skill routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.

Routing rules:
- Sentry issue lookup, "check Sentry", "what's the error for issue <id>" → invoke sentry
- GitHub issue work, "solve issue #N", "work on this issue", "implement the fix for issue X" → invoke solve-issue
