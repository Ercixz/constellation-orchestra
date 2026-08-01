# Connector：AI Agent 接入（agents/）

> AI Agent = 真正执行具体任务的单元。本目录按 agent 分文件，说明各自**供应商配置、皮肤/设置、如何被编排器拉起**。
> 接入总览与可插拔原则见 `connectors/agents/` 各文件；哲学见 `philosophy.md`。

## 本目录内容

| 文件 | Agent | CLI | 核心配置 |
|---|---|---|---|
| `claude-code.md` | Claude Code | `claude` | Anthropic API key / base URL；原生皮肤 |
| `codex.md` | Codex | `codex` | OpenAI 登录 / API key；fast mode |
| `opencode.md` | OpenCode | `opencode` | provider 配置（API key/base URL/model）；TUI 皮肤 |

## 共同原则（可插拔）

- Agent 只是执行单元：被派单、干活、回传，然后结束，不绑定任何编排器。
- 换 agent 不换编排器，换编排器不换 agent（详见 `philosophy.md` 二元论）。
- 供应商凭据只进本机 600 权限配置，绝不入文档/回执明文。

## 被编排器拉起的三种方式（通用）

| 编排器 | 拉起方式 |
|---|---|
| **Orca** | `worktree create --name <n> --repo R --agent <id> --prompt "<任务>"`；或 `terminal create --worktree active --command "claude|codex|opencode"` |
| **CMUX** | `new-surface --type agent-session --provider codex|claude|opencode`；或 surface 内直接跑 CLI + `cmux send` 派单 |
| **Paseo** | `create_agent {title, provider, initialPrompt}`；`send_agent_prompt {agentId, prompt}` |

各 agent 的供应商/皮肤细节见对应文件。

---

相关文件：`philosophy.md`、`connectors/orchestrators/*.md`（编排器侧）。
