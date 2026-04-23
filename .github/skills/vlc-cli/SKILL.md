---
name: vlc-cli
description: Use when the user asks to control VLC through command line/RC/Telnet, needs task-to-command mapping (playback, seek, tracks, speed, volume, queue), or needs troubleshooting/safe execution constraints for VLC CLI workflows.
---

# VLC CLI / RC 操作指引

用于指导智能体在 VLC 命令行与 RC 交互下进行稳定、可控、可恢复的媒体操作。

## 1) 适用触发场景

当用户出现以下需求时应加载本 Skill：

- “用命令行控制 VLC 播放/暂停/跳转/切歌”
- “通过 RC/Telnet 精准控制 VLC”
- “给我 VLC 命令映射表”
- “VLC CLI 命令无响应，如何排查”
- “让 agent 自动化执行播放队列操作”

## 2) 模式边界与选择

优先级：**RC 交互控制** > 一次性 CLI 参数。

### A. 一次性 CLI 参数模式

适合：启动即播放、固定参数播放、无需持续交互。

- 常见启动参数：`--intf dummy`、`--play-and-exit`、`--fullscreen`、`--start-time`
- 不适合频繁动态操作（如连续 seek、切换轨道、实时队列管理）

### B. RC（远程控制）模式

适合：持续会话控制、状态查询、自动化步骤。

- 本地 socket 常见形式：`--extraintf rc --rc-unix <socket>`
- 网络控制常见形式：`--extraintf rc --rc-host <host:port>`
- 核心原则：先确认连接可用，再执行操作；每次关键动作后做状态回读

### C. 接口选择提示（Lua CLI / Telnet / oldrc）

- `--extraintf rc` 通常加载 RC/CLI 接口，但不同版本会落在 Lua CLI 或 oldrc
- Lua CLI 常见入口：`--rc-host <host:port>`、`--cli-host <source>`
- Lua Telnet 常见入口：`--telnet-host`、`--telnet-port`、`--telnet-password`
- oldrc 可见选项：`--rc-show-pos`、`--rc-quiet`、`--rc-host <host:port>`
- 版本差异较大时，必须以运行时 `vlc --help` / `vlc -H` 与 RC `help` 为准

## 3) 标准操作流程（Agent 执行顺序）

1. **预检**：确认 VLC 可执行、目标媒体路径/URL 有效、控制端点可达  
2. **建立会话**：启动 VLC 或连接现有 RC 端点  
3. **能力探测**：优先读取 `help` / `status`（不同版本命令差异）  
4. **执行命令**：按“单步动作 + 状态确认”方式推进  
5. **异常恢复**：超时/无响应时重连并重试最小必要步骤  
6. **收尾**：按用户意图停止、退出或保持会话

## 4) 任务到命令映射（常用）

> 说明：具体命令名可能因 VLC 版本/平台略有差异，执行前应用 `help` 确认。

| 任务 | 常用命令 |
|---|---|
| 添加并立即播放 | `add <path-or-url>` |
| 加入队列（不打断当前） | `enqueue <path-or-url>` |
| 播放 / 暂停 | `play` / `pause` |
| 停止 | `stop` |
| 下一首 / 上一首 | `next` / `prev` |
| 跳转（绝对/相对） | `seek <value>`（如秒或百分比，按版本支持） |
| 查询状态 | `status`、`info` |
| 查询时长与进度 | `get_length`、`get_time` |
| 倍速 | `faster`、`slower`、`normal` |
| 音量 | `volume <value>`、`volup`、`voldown` |
| 音轨切换 | `atrack <id>` |
| 字幕轨切换 | `strack <id>` |
| 播放列表查看/定位 | `playlist`（并结合条目 id 决策） |
| 退出 | `quit` |

## 4.0) CLI 帮助与能力探测（必须掌握）

| 目的 | 命令 |
|---|---|
| 基础帮助 | `vlc --help` |
| 完整帮助（更长） | `vlc -H` |
| 模块帮助 | `vlc -p <module> --advanced --help-verbose` |

说明：Windows 上 `vlc --help` 会生成帮助文本文件；输出过长时优先使用 `-H` 或模块级帮助。

## 4.1) 指令对照补充（按类别）

> 与官方命令行帮助页对照时，建议按“启动参数 / RC 会话命令”两层核对。  
> 不同版本命令可用性可能不同，统一以运行时 `vlc --help`、`vlc --longhelp`、RC `help` 输出为最终依据。

### A. 启动与接口参数（CLI）

| 目标 | 常见参数 |
|---|---|
| 查看帮助 | `--help`、`--longhelp` |
| 列出模块/能力 | `--list` |
| 指定界面 | `--intf <name>`（如 `dummy`、`rc`） |
| 叠加 RC 接口 | `--extraintf rc` |
| RC 网络端点 | `--rc-host <host:port>` |
| RC 本地 socket | `--rc-unix <path>` |
| CLI/Telnet 接口 | `--cli-host <source>`、`--telnet-host`、`--telnet-port`、`--telnet-password` |
| RC 行为 | `--rc-quiet`、`--rc-show-pos` |
| 播放后退出 | `--play-and-exit` |
| 起播时间 | `--start-time <sec>` |
| 全屏起播 | `--fullscreen` |

### A.1) 日志与静默（CLI）

| 目标 | 常见参数 |
|---|---|
| 静默模式 | `-q` / `--quiet` |
| 日志写文件 | `--file-logging`、`--logfile <path>` |
| 日志格式 | `--logmode {text,html}` |
| 日志级别 | `--log-verbose { -1,0,1,2,3 }` |

### B. RC 会话命令（控制面）

| 类别 | 常见命令 |
|---|---|
| 基础控制 | `play`、`pause`、`stop`、`next`、`prev` |
| 定位与进度 | `seek <value>`、`get_time`、`get_length` |
| 媒体装载 | `add <mrl>`、`enqueue <mrl>` |
| 列表管理 | `playlist`、`clear` |
| 状态查询 | `status`、`info`、`stats` |
| 播放模式 | `repeat`、`loop`、`random` |
| 倍速控制 | `faster`、`slower`、`normal` |
| 音量控制 | `volume <value>`、`volup`、`voldown` |
| 轨道控制 | `atrack <id>`、`vtrack <id>`、`strack <id>` |
| 章节/标题 | `chapter <id>`、`title <id>` |
| 会话退出 | `quit` |

## 5) 常见失败场景与恢复策略

| 场景 | 判定信号 | 恢复策略 |
|---|---|---|
| RC 连接失败 | 端口/Socket 不可达、连接超时 | 校验启动参数；确认端点；重建 VLC 会话后重连 |
| 命令无响应 | 命令发送后无状态变化 | 先 `status`；若无回包则重连；仅重放最后必要命令 |
| 媒体不可用 | `add/enqueue` 后报错或不进入播放 | 校验路径、权限、URL 可访问性；先替换为可用样例验证链路 |
| 跨平台差异 | 命令存在但行为不一致 | 强制执行 `help` 能力探测；按平台条件分支 |
| 队列状态混乱 | `playlist` 与预期不符 | 先读取 playlist 再决策，不盲目 next/prev 连续操作 |

## 6) 安全与约束

- **先查后改**：关键操作前后都读取状态，避免盲执行  
- **最小破坏原则**：未获明确许可，不执行 `clear`、批量删除或 `quit`  
- **输入安全**：路径/URL 参数禁止拼接不可信 shell 片段  
- **幂等优先**：尽量使用可重复执行且副作用可控的步骤  
- **用户意图保护**：涉及停止播放、切换媒体、退出会话需与用户目标一致

## 6.1) 冷门/高级参数处理原则

- 优先在 [reference/cli.txt](reference/cli.txt) 中定位对应模块与参数，再给出精确选项名
- 若用户请求属于“视觉效果、视频输出、编码、滤镜、音频重采样”等高级模块，先用 `vlc -p <module> --advanced --help-verbose` 做二次确认
- 仅在参考里存在的选项才可下结论；不确定时明确提示“需以当前 VLC 版本输出为准”

## 7) 验收标准（使用本 Skill 时）

- 能根据用户请求稳定映射为可执行 VLC CLI/RC 动作
- 每个关键动作都包含状态确认与结果反馈
- 失败时给出可执行恢复路径，而非仅报错
- 输出包含边界说明（版本/平台差异）与安全约束

## 8) 典型请求验收样例（用于稳定性核对）

- “把 A 加到队列并在当前播放结束后切换”  
  - 期望：先 `status`，再 `enqueue`，再 `playlist` 回读确认
- “跳到 01:30 并把音量调到 200”  
  - 期望：执行 `seek` 与 `volume`，随后 `get_time` / `status` 校验
- “切到下一条字幕轨并确认是否生效”  
  - 期望：执行 `strack <id>`，随后 `info` 或 `status` 回读
- “连接 RC 失败怎么办”  
  - 期望：返回端点检查、重连顺序、最小重放命令方案
