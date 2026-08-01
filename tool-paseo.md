# Paseo 命令速查（详）

> 总览索引见 `SKILL.md`。本文件是 Paseo 工具/CLI 双面提炼。

## 启动 / 前提

- daemon：`paseo daemon status`；健康：`curl -s localhost:6767/api/health`
- CLI 不在 PATH：macOS `/Applications/Paseo.app/Contents/Resources/bin/paseo`；首跑 `installCli` hook 会 symlink 到 `~/.local/bin/paseo` 并加 PATH。若未生效，提示用户 symlink，别静默做。
- 用任何 Paseo skill 前**真实读取** `~/.paseo/orchestration-preferences.json`：`providers`（role→provider 映射，直接传给 `create_agent`）+ `preferences`（自由文本，织入 prompt）。缺失时用合理默认并告知一次。Categories：`impl`/`ui`/`research`/`planning`/`audit`。

## CLI 速查

```bash
paseo workspace create --isolation worktree --mode branch-off --new-branch fix-x --base main
paseo workspace create --isolation worktree --mode checkout-branch --branch existing-work
paseo workspace create --isolation worktree --mode checkout-pr --pr-number 42
paseo run --provider codex/gpt-5.4 --mode full-access --workspace <id> "<prompt>"
paseo run --provider codex/gpt-5.4 --mode full-access --new-workspace worktree --worktree-mode branch-off --new-branch fix-x --base main "<prompt>"
paseo send <agent-id> "<follow-up>"
paseo ls
paseo script ls|start|stop <name> [--cwd <path> | --workspace <id>]
paseo schedule create --cron "*/15 * * * *" "ping main build"
paseo heartbeat create --cron "*/15 * * * *" "check the build"
```

发现命令：`paseo --help`、`paseo <cmd> --help`。

## MCP 工具面

### Workspace
- `create_workspace`：必填 `isolation`（`local`|`worktree`）。worktree 支持 `mode: "branch-off"|"checkout-branch"|"checkout-pr"`；`branchName`/`baseBranch` 新建分支，`branch` 用现有分支，`prNumber`（+可选 `forge`/`projectPath`）用 change request；`worktreeSlug` 控托管路径。返回 `workspaceId`。
- `list_workspaces`、`archive_workspace`（`{workspaceId}`；归档 workspace+agents+terminals；本地目录保留，owned worktree 仅当最后引用归档后移除）。

### 工作区脚本（paseo.json 配置）
- `list_workspace_scripts`、`start_workspace_script`、`stop_workspace_script`（均为 `{workspaceId[, scriptName]}`）。
- CLI：`paseo script ls|start|stop <name>`。

### Agents
- `create_agent`：必填 `title`/`provider`/`initialPrompt`；可选 `workspaceId`/`notifyOnFinish`/`settings`/`labels`。运行设置放 `settings`：`modeId`、`thinkingOptionId`、provider `features`（Codex fast：`{features:{"fast_mode":true}}`）。返回 `{agentId, workspaceId}`。Agent-scoped 创建即你的 subagent，省略 `workspaceId` 用当前 workspace。`notifyOnFinish` 默认 true（仅真 fire-and-forget 设 false）。
- `send_agent_prompt`：`{agentId, prompt}`。Agent-scoped 默认 `background:true`+`notifyOnFinish:true`；顶层默认阻塞无回调；同步用 `background:false` 取返回值。
- `update_agent`：`{agentId, name?, labels?, settings?}`，settings 可改 `modeId`/`model`/`thinkingOptionId`/`features`。
- `list_agents`：filter `cwd`/`statuses`/`sinceHours`/`includeArchived`；`archive_agent`：`{agentId}`（中断+移出列表）。

### Provider
- `list_providers`（紧凑）、`list_models`（大，按需）、`inspect_provider`（`provider` 必填；非 agent 会话传 `cwd`；可选 `settings` draft）。只设置返回的 feature id。

### Schedules / Heartbeats
- `create_schedule`：必填 `prompt`/`cron`/`provider`；可选 `timezone`/`name`/`cwd`/`maxRuns`/`expiresIn`。用定时新 agent。
- `create_heartbeat`：必填 `prompt`/`cron`；可选同上。用提醒/盯构建/状态检查（回到本会话）。
- `delete_heartbeat` 停止。无 heartbeat update 工具——改任务/频率就删了重建。Schedule 有完整 list/inspect/update/pause/resume/run-once/log/delete。

### Models
`claude/sonnet`（默认）、`claude/opus`（难推理）、`codex/gpt-5.4`（前沿编码）、`claude/haiku`（仅测试）。

## 等待哲学

- 任务 10–30+ 分钟是常态，偏好异步。
- 不轮询 `list_agents`/`get_agent_status`；完成/出错/要权限时通知自行到达。
- `notifyOnFinish` 留空或 `true`，除非真 fire-and-forget。

## Ops 调试

| 项 | 默认 |
| --- | --- |
| Listen | `127.0.0.1:6767`（`PASEO_LISTEN` 覆盖） |
| Home | `~/.paseo`（`PASEO_HOME` 覆盖） |
| Daemon log | `$PASEO_HOME/daemon.log` |
| Agent state | `$PASEO_HOME/agents/<id>.json` |
| Worktrees | `$PASEO_HOME/worktrees/`（或 `worktrees.root`） |
| PID | `$PASEO_HOME/paseo.pid` |

调试顺序：1) `tail -n 200 ~/.paseo/daemon.log` → 2) `paseo daemon status` → 3) `curl -s localhost:6767/api/health`。

**绝不未经用户批准重启 daemon**——会杀掉所有运行中 agent（含提问者自己）。
