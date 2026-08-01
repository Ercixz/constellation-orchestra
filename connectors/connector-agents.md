# Connector：如何连接 AI Agent

> AI Agent = 真正执行具体任务的单元。接入本 skill 的三个 agent：Claude Code、Codex、OpenCode。
> 本文件是"接入总表"——怎么连、怎么被编排器拉起、可插拔原则。

## 一、三 Agent 接入总表

| Agent | CLI | 说明 |
|---|---|---|
| **Claude Code** | `claude` | Anthropic 官方 CLI agent，擅长长上下文/复杂推理 |
| **Codex** | `codex` | OpenAI 编码 agent，有 fast mode（`{"features":{"fast_mode":true}}`） |
| **OpenCode** | `opencode` | 本场主用 agent；TUI 需 ≥60 列；模型可命令行指定（如 `-m deepseek/deepseek-v4-flash`） |

> 共性：**三者都是纯 CLI 程序**，被编排器当作子进程/子会话拉起，本身不持有"场"。

## 二、如何被编排器拉起

### Orca（worktree + 内置编排）
- `orca worktree create --name <n> --repo <R> --agent <id> --prompt "<任务>"`：在隔离 worktree 里直接起 agent。
- `orca terminal create --worktree active --command "codex"`：在当前 worktree 起指定 agent。
- Orca 的 `orchestration dispatch` / `worker-start` 会把任务派给对应 agent。

### CMUX（surface / 面板）
- `new-surface --type agent-session --provider codex|claude|opencode`：直接起 agent 会话 surface。
- 或任意 surface 内 `opencode -m <model>` 起 TUI，然后 `cmux send --surface <UUID>` 派单。

### Paseo（create_agent）
- `create_agent {title, provider, initialPrompt}`：创建受守护的子 agent。
- `send_agent_prompt {agentId, prompt}`：后续跟进；`background:true`+`notifyOnFinish:true` 异步等回传。
- `paseo run --provider codex/gpt-5.4 --mode full-access --workspace <id> "<prompt>"`：CLI 快捷跑。
- provider 映射与偏好见 `~/.paseo/orchestration-preferences.json`（role→provider + 自由文本偏好）。

## 三、可插拔原则

- **Agent 只是执行单元**：被派单、干活、回传，然后结束。不绑定任何编排器。
- **换 agent 不换编排器**：同一个 worktree / surface / Paseo agent 位，随时换 `codex`→`opencode`→`claude`。
- **换编排器不换 agent**：同一个 agent 品牌可以被 Orca、CMUX、Paseo 任一拉起。
- 分类判据（见 philosophy.md）：**会"组织/调度"的是编排器，会"执行任务"的是 Agent**。`claude`/`codex`/`opencode` 永远是后者。

## 四、接入检查清单

1. 确认 CLI 在 PATH（`claude`/`codex`/`opencode`），或用全路径。
2. 确认账号/认证（`orca account`、codex 登录、opencode 配置）。
3. 拉起后先发一条冒烟消息验证能通，再正式派单。
4. 无凭据明文；API key 只进本机 600 权限配置。

---

相关文件：`connectors/connector-orchestrators.md`（编排器接入）、`philosophy.md`（二元论与可插拔）、`orchestrators/*.md`（各编排器如何拉起 agent）。
