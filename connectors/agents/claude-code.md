# Agent：Claude Code

> 分类：**AI Agent**（执行单元）。Anthropic 官方 CLI agent。
> 总览见 `connectors/agents/README.md`；哲学见 `philosophy.md`。

## CLI 连接

- 命令：`claude`
- 前提：已安装（brew / 官方脚本）；已认证（`claude login` 或 ANTHROPIC_API_KEY）

## 供应商配置

- **API key**：`ANTHROPIC_API_KEY` 环境变量，或 `claude login`（OAuth，可走团队订阅额度）。
- **Base URL**：若走自定义网关/代理，设 `ANTHROPIC_BASE_URL`（如 LiteLLM 网关 `https://litellm.isonoimic.site`）。
- 模型：`claude` 默认取账号可用模型（opus/sonnet/haiku）；Paseo 里对应 `claude/sonnet`、`claude/opus`、`claude/haiku`。
- ⚠️ 凭据只进本机 600 权限配置（`~/.claude/.credentials.json` 等），绝不入文档明文。

## 皮肤 / 设置

- 交互皮肤由终端主题决定（如 CMUX surface 内跑 `claude`，继承 Ghostty 主题）。
- `claude config` 可调偏好在 `~/.claude.json` / `~/.claude/settings.json`。
- 与 OpenCode 类似是 TUI：窄面板需足够列宽。

## 如何被编排器拉起

| 编排器 | 拉起方式 |
|---|---|
| **Orca** | `worktree create --name <n> --repo R --agent claude --prompt "<任务>"`；或 `terminal create --worktree active --command "claude"` |
| **CMUX** | `new-surface --type agent-session --provider claude`；或 surface 内 `claude` + `cmux send --surface <UUID> "..."` 派单 |
| **Paseo** | `create_agent {provider: "claude/sonnet", title, initialPrompt}`；`send_agent_prompt {agentId, prompt}` |

## 可插拔

- Claude 只是执行单元；换 Codex/OpenCode 不改变编排器侧任何配置。
- 适合长上下文/复杂推理；对响应速度要求高的短任务可换轻量 agent。

---

相关文件：`connectors/agents/README.md`、`connectors/agents/codex.md`、`connectors/agents/opencode.md`。
