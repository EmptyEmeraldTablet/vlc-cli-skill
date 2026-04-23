---
name: vlc-cli
description: Use when the user asks to control VLC through command line/RC/Telnet, needs task-to-command mapping (playback, seek, tracks, speed, volume, queue), or needs troubleshooting/safe execution constraints for VLC CLI workflows.
---

# VLC CLI / RC Operations Guide

Guidance for agents to perform stable, controllable, and recoverable media operations via VLC command line and RC interaction.

## 1) When to Apply

Load this skill when the user asks for:

- Control of VLC playback via command line (play/pause/seek/next)
- Precise RC or Telnet control
- A VLC command mapping table
- Troubleshooting when VLC CLI commands do not respond
- Automated playlist or queue operations

## 2) Mode Boundaries and Selection

Priority: **RC interactive control** > one-shot CLI parameters.

### A. One-shot CLI Parameters

Best for: start-and-play, fixed parameters, no continuous interaction.

- Common startup flags: `--intf dummy`, `--play-and-exit`, `--fullscreen` (verify additional flags via `vlc --help`)
- Not suitable for frequent dynamic operations (continuous seek, track switching, live queue management)

### B. RC (Remote Control) Mode

Best for: ongoing session control, status queries, automation steps.

- Local socket form: (not in reference/cli.txt; validate via `vlc --help` on the target build)
- Network endpoint form: `--extraintf rc --rc-host <host:port>`
- Core principle: confirm the connection before acting; read back state after each critical action

### C. Interface Selection Notes (Lua CLI / Telnet / oldrc)

- `--extraintf rc` usually loads RC/CLI, but versions can map to Lua CLI or oldrc
- Lua CLI entry points: `--rc-host <host:port>`, `--cli-host <source>`
- Lua Telnet entry points: `--telnet-host`, `--telnet-port`, `--telnet-password`
- oldrc options: `--rc-show-pos`, `--rc-quiet`, `--rc-host <host:port>`
- For large version differences, trust runtime `vlc --help` / `vlc -H` and RC `help`

## 3) Standard Operating Flow (Agent Order)

1. **Pre-check**: confirm VLC executable, media path/URL validity, and reachable control endpoint
2. **Establish session**: start VLC or connect to an existing RC endpoint
3. **Capability probe**: read `help` / `status` first (commands vary by version)
4. **Execute commands**: move in single-step actions with state confirmation
5. **Recovery**: on timeout/no response, reconnect and retry the minimum necessary steps
6. **Wrap-up**: stop, quit, or keep the session aligned to user intent

## 4) Task-to-Command Mapping (Common)

> RC session commands are not listed in reference/cli.txt; always confirm via RC `help` in the running build.
> Note: command names can vary across versions and platforms. Confirm with `help` before executing.

| Task | Common Command |
|---|---|
| Add and play immediately | `add <path-or-url>` |
| Enqueue without interrupting | `enqueue <path-or-url>` |
| Play / Pause | `play` / `pause` |
| Stop | `stop` |
| Next / Previous | `next` / `prev` |
| Seek (absolute/relative) | `seek <value>` (seconds or percent, version-dependent) |
| Query status | `status`, `info` |
| Query length and time | `get_length`, `get_time` |
| Playback speed | `faster`, `slower`, `normal` |
| Volume | `volume <value>`, `volup`, `voldown` |
| Audio track switch | `atrack <id>` |
| Subtitle track switch | `strack <id>` |
| Playlist view/locate | `playlist` (use item ids to decide) |
| Quit session | `quit` |

## 4.0) CLI Help and Capability Probing (Required)

| Purpose | Command |
|---|---|
| Basic help | `vlc --help` |
| Full help (longer) | `vlc -H` |
| Module help | `vlc -p <module> --advanced --help-verbose` |

Note: On Windows, `vlc --help` creates a help text file; if output is too long, prefer `-H` or module-level help. You can page the help file with:

```
more "%PROGRAMFILES%\VideoLAN\VLC\vlc-help.txt"
```

## 4.1) Command Reference by Category

> When cross-checking with official CLI help, validate in two layers: startup parameters and RC session commands.  
> Availability can differ by version; runtime `vlc --help`, `vlc --longhelp`, and RC `help` are authoritative.

### A. Startup and Interface Parameters (CLI)

| Goal | Common Parameters |
|---|---|
| Show help | `--help`, `--longhelp` |
| List modules/capabilities | `--list` |
| Select interface | `--intf <name>` (e.g., `dummy`, `rc`) |
| Add RC interface | `--extraintf rc` |
| RC network endpoint | `--rc-host <host:port>` |
| RC local socket | `--rc-unix <path>` |
| CLI/Telnet interface | `--cli-host <source>`, `--telnet-host`, `--telnet-port`, `--telnet-password` |
| RC behavior | `--rc-quiet`, `--rc-show-pos` |
| Exit after playback | `--play-and-exit` |
| Start time | (not in reference/cli.txt; verify via `vlc --help`) |
| Fullscreen start | `--fullscreen` |

### A.1) Logging and Quiet Mode (CLI)

| Goal | Common Parameters |
|---|---|
| Quiet mode | `-q` / `--quiet` |
| Log to file | `--file-logging`, `--logfile <path>` |
| Log format | `--logmode {text,html}` |
| Log verbosity | `--log-verbose { -1,0,1,2,3 }` |

### B. RC Session Commands (Control Surface)

| Category | Common Commands |
|---|---|
| Basic control | `play`, `pause`, `stop`, `next`, `prev` |
| Seek and progress | `seek <value>`, `get_time`, `get_length` |
| Media loading | `add <mrl>`, `enqueue <mrl>` |
| Playlist management | `playlist`, `clear` |
| Status queries | `status`, `info`, `stats` |
| Playback modes | `repeat`, `loop`, `random` |
| Speed control | `faster`, `slower`, `normal` |
| Volume control | `volume <value>`, `volup`, `voldown` |
| Track control | `atrack <id>`, `vtrack <id>`, `strack <id>` |
| Chapters/titles | `chapter <id>`, `title <id>` |
| Session exit | `quit` |

## 5) Common Failure Scenarios and Recovery

| Scenario | Signal | Recovery |
|---|---|---|
| RC connection failure | Port/socket unreachable, timeout | Validate startup params, confirm endpoint, recreate VLC session, then reconnect |
| Commands no response | No state change after sending | Run `status`; if no reply, reconnect; replay only the last necessary command |
| Media unavailable | Errors after `add`/`enqueue` or no playback | Validate path, permissions, and URL; swap in a known-good sample to verify chain |
| Cross-platform differences | Command exists but behaves differently | Force capability probe with `help`, branch by platform |
| Queue state mismatch | `playlist` differs from expectation | Read playlist before deciding; avoid blind rapid `next`/`prev` |

## 6) Safety and Constraints

- **Read before write**: read state before and after critical actions to avoid blind execution
- **Minimal disruption**: do not run `clear`, bulk deletes, or `quit` without explicit permission
- **Input safety**: never concatenate untrusted shell fragments into path/URL arguments
- **Idempotence first**: prefer repeatable steps with controlled side effects
- **Protect user intent**: stopping playback, switching media, or quitting must align with user intent

## 6.1) Handling Rare/Advanced Parameters

- Prefer locating the exact module and option in [reference/cli.txt](reference/cli.txt) before answering
- For advanced areas (visual effects, video output, encoding, filters, audio resampling), verify with `vlc -p <module> --advanced --help-verbose` first
- Only assert options that exist in the reference; otherwise state that the current VLC version output is authoritative

## 7) Acceptance Criteria (When Using This Skill)

- Map user requests to executable VLC CLI/RC actions reliably
- Include state confirmation and feedback after each key action
- Provide executable recovery steps instead of only errors
- Include boundary notes (version/platform differences) and safety constraints

## 8) Example Requests and Expected Checks

- "Enqueue A and switch after current playback"
  - Expected: `status`, then `enqueue`, then `playlist` readback
- "Seek to 01:30 and set volume to 200"
  - Expected: `seek` and `volume`, then `get_time` / `status` verification
- "Switch to next subtitle track and confirm"
  - Expected: `strack <id>`, then `info` or `status` readback
- "RC connection failed, what now?"
  - Expected: endpoint checks, reconnect order, minimal replay plan
