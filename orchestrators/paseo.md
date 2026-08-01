# Paseo — 编排器档案

> 分类：**编排器**（agent 生命周期守护、定时/心跳、跨会话的"场"）。
> 接入总表见 `connectors/connector-orchestrators.md`；哲学见 `philosophy.md`。

## 编排器四问

### 1. 用什么 CLI/界面连接它

- CLI：`~/.local/bin/paseo`（daemon 侧）；macOS 备用 `/Applications/Paseo.app/Contents/Resources/bin/paseo`。
- 前提：**需 daemon 运行**。健康：`paseo daemon status`；`curl -s localhost:6767/api/health`。
- 首跑 `installCli` hook 会 symlink 到 `~/.local/bin/paseo` 并加 PATH；若未生效，提示用户 symlink，别静默做。
- 用任何 Paseo skill 前**真实读取** `~/.paseo/orchestration-preferences.json`：`providers`（role→provider 映射）+ `preferences`（自由文本，织入 prompt）。缺失时用合理默认并告知一次。
- **绝不未经用户批准重启 daemon**——会杀掉所有运行中 agent（含提问者自己）。

### 2. 如何沟通、协作、连接到一起

- **发消息**：`paseo send <agent-id> "<follow-up>"`（核心回传/跟进通道）。
- **创建 agent**：`create_agent {title, provider, initialPrompt}`；`send_agent_prompt {agentId, prompt}`（`background:true`+`notifyOnFinish:true` 异步等回传）。
- **CLI 快捷跑**：`paseo run --provider codex/gpt-5.4 --mode full-access --workspace <id> "<prompt>"`。
- **协作骨架**：worker 干完 `paseo send <manager-id> --no-wait "..."` → Manager 的 Paseo 会话自动收到 → 验收归档。
- 等待哲学：任务 10–30+ 分钟是常态，**不轮询** `list_agents`/`get_agent_status`；完成/出错/要权限时通知自行到达。
- 与 Orca/CMUX 互连：见 `connectors/connector-orchestrators.md`（CMUX 派单、Orca 子 Agent）。

### 3. 如何在各空间内理解空间组织结构

Paseo 的空间结构：

| 层 | 空间单元 | 说明 |
|---|---|---|
| 顶层 | Workspace | 隔离执行环境（local / worktree，可选） |
| 中层 | Agent | 一个受守护的子 agent（`agents/<id>.json`） |
| 底层 | Terminal / Script | 工作区脚本、终端会话 |

- Workspace：`create_workspace {isolation: local|worktree}`（worktree 支持 `branch-off`/`checkout-branch`/`checkout-pr`）；`list_workspaces`、`archive_workspace`。
- Agent：`create_agent`（必填 `title`/`provider`/`initialPrompt`；可选 `workspaceId`/`notifyOnFinish`/`settings`/`labels`）；`list_agents`（filter `cwd`/`statuses`/`sinceHours`/`includeArchived`）；`archive_agent`。
- 脚本：`start_workspace_script` / `stop_workspace_script` / `list_workspace_scripts`；CLI `paseo script ls|start|stop`。
- 定时：`create_schedule {prompt, cron, provider}`（定时新 agent）；`create_heartbeat {prompt, cron}`（回本会话的提醒/盯构建）。

### 4. 编排 GUI 关系（多开窗口/切分 pane/布局）

- Paseo 本体是 **daemon（无 GUI 面板）**——它是"后台守门人"，不管理窗口/面板。
- 多开/布局交给编排层其他工具：想并排看多个 agent → 用 CMUX surface；想隔离开发 → 用 Orca worktree。
- Paseo 的"GUI 关系" = **agent 生命周期视图**：`paseo ls` 看全部 agent、`daemon log` 看运行记录、`get_agent_status` 查状态。
- 即：**Paseo 管"谁在跑、跑没跑完、怎么跟进"，CMUX/Orca 管"长什么样、摆哪"**。

## 命令速查（详）

### CLI 速查

```bash
paseo workspace create --isolation worktree --mode branch-off --new-branch fix-x --base main
paseo workspace create --isolation worktree --mode checkout-branch --branch existing-work
paseo workspace create --isolation worktree --mode checkout-pr --pr-number 42
paseo run --provider codex/gpt-5.4 --mode full-access --workspace <id> "<prompt>"
paseo send <agent-id> "<follow-up>"
paseo ls
paseo script ls|start|stop <name> [--cwd <path> | --workspace <id>]
paseo schedule create --cron "*/15 * * * *" "ping main build"
paseo heartbeat create --cron "*/15 * * * *" "check the build"
```

发现命令：`paseo --help`、`paseo <cmd> --help`。

### MCP 工具面

- **Workspace**：`create_workspace`（`isolation` 必填；worktree 支持 `mode`/`branchName`/`baseBranch`/`branch`/`prNumber`/`worktreeSlug`）；`list_workspaces`、`archive_workspace`。
- **Agents**：`create_agent`（`title`/`provider`/`initialPrompt` 必填；`settings.modeId`、`thinkingOptionId`、provider `features`（Codex fast：`{"features":{"fast_mode":true}}`）；返回 `{agentId, workspaceId}`）；`send_agent_prompt`（Agent-scoped 默认 `background:true`+`notifyOnFinish:true`）；`update_agent`；`list_agents`、`archive_agent`。
- **Provider**：`list_providers`（紧凑）、`list_models`（大，按需）、`inspect_provider`（只设置返回的 feature id）。
- **Schedules / Heartbeats**：`create_schedule`（必填 `prompt`/`cron`/`provider`）；`create_heartbeat`（必填 `prompt`/`cron`）；`delete_heartbeat` 停止；Schedule 有完整 list/inspect/update/pause/resume/run-once/log/delete。
- **Models**：`claude/sonnet`（默认）、`claude/opus`（难推理）、`codex/gpt-5.4`（前沿编码）、`claude/haiku`（仅测试）。

## 等待哲学

- 任务 10–30+ 分钟是常态，偏好异步。
- 不轮询；完成/出错/要权限时通知自行到达。
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

---

相关文件：`philosophy.md`、`connectors/connector-orchestrators.md`、`connectors/connector-agents.md`、`SKILL.md`。
