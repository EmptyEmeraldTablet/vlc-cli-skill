# vlc-cli-skill

Repository for a GitHub Copilot Skill that guides agents to operate VLC via CLI/RC safely and reliably.

## Essential commands

- No build/test/lint pipeline is currently configured in this repository.

## Repository structure

- `.github/skills/` — on-demand skill definitions
- `README.md` — minimal repository readme
- `AGENTS.md` — root context for agents
- `CLAUDE.md` — Claude bridge to AGENTS and skills

## Skills

| Skill | When to use |
|---|---|
| [vlc-cli](.github/skills/vlc-cli/SKILL.md) | When the user wants precise VLC CLI/RC control, command mapping, troubleshooting, and safe operation constraints. |

## Key entry points

- VLC operational guidance: `.github/skills/vlc-cli/SKILL.md`
