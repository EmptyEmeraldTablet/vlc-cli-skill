# vlc-cli-skill

GitHub Copilot Skill repository for guiding agents to operate VLC safely and reliably via CLI/RC/Telnet.

## What this repo contains

- A curated VLC CLI/RC skill with task-to-command mappings and safety rules.
- A command reference snapshot used to validate uncommon options.
- Agent entry points for integrations that load skills on demand.

## Repository structure

- [.github/skills/vlc-cli/SKILL.md](.github/skills/vlc-cli/SKILL.md) - main skill content
- [reference/cli.txt](reference/cli.txt) - VLC command-line help snapshot
- [AGENTS.md](AGENTS.md) - root agent context
- [CLAUDE.md](CLAUDE.md) - bridge for Claude-based workflows

## How to use the skill

1. Load the skill file when user requests involve VLC CLI/RC control.
2. Prefer RC/Telnet for interactive sessions; use one-shot CLI for simple playback.
3. Confirm capabilities with `vlc --help` / `vlc -H` and RC `help` before issuing commands.

See the full operational guidance in [.github/skills/vlc-cli/SKILL.md](.github/skills/vlc-cli/SKILL.md).

## Reference

The [reference/cli.txt](reference/cli.txt) file is the authoritative snapshot for uncommon options and module-level flags. Use it to validate rare requests before proposing commands.

## Notes

- No build, test, or lint pipeline is configured in this repository.
