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

## 7) 验收标准（使用本 Skill 时）

- 能根据用户请求稳定映射为可执行 VLC CLI/RC 动作
- 每个关键动作都包含状态确认与结果反馈
- 失败时给出可执行恢复路径，而非仅报错
- 输出包含边界说明（版本/平台差异）与安全约束
