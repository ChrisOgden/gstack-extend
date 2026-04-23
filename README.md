# gstack-extend

Personal extension skills layered on top of [gstack](https://github.com/ChrisOgden/gstack) and [gstack-extend](https://github.com/ChrisOgden/gstack-extend).

| Skill | What it does |
|-------|--------------|
| `/sentry` | Look up a Sentry issue by ID and display error details with stacktrace |
| `/solve-issue` | Read a GitHub issue from a repo and use it as the prompt to implement a fix |

## Installation

```bash
git clone git@github.com:ChrisOgden/gstack-extend.git ~/.claude/skills/gstack-extend
~/.claude/skills/gstack-extend/setup
```

Installs all skills into `~/.claude/skills/` as symlinks back to this repo.
To uninstall: `~/.claude/skills/gstack-extend/setup --uninstall`

## Versioning

4-digit format: `MAJOR.MINOR.PATCH.MICRO`. Source of truth: `VERSION` file.
Tags are created automatically on merge to main when `VERSION` changes.

## License

[MIT](LICENSE)
