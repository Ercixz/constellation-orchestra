# Agent：Codex

> 分类：**AI Agent**（执行单元）。OpenAI 编码 agent。
> 总览见 `connectors/agents/README.md`；哲学见 `philosophy.md`。

## CLI 连接

- 命令：`codex`
- 前提：已安装（brew / 官方安装）；已登录（`codex login`，或 `OPENAI_API_KEY`）

## 供应商配置

- **认证**：`codex login`（OAuth，走 ChatGPT 订阅额度）或 `OPENAI_API_KEY` 环境变量。
- **Base URL**：若走网关/代理，设 `OPENAI_BASE_URL`。
- **fast mode**：Paseo 里通过 provider features 开快模式 `{"features":{"fast_mode":true}}`，适合快速小改。
- **模型**：Paseo 常用 `codex/gpt-5.4`（前沿编码）。
- ⚠️ 凭据只进本机 600 权限配置，绝不入文档明文。

## 皮肤 / 设置

- Codex CLI 是终端界面；`codex` 在窄面板里同样需足够列宽。
- 偏好/认证文件在 `~/.codex/`；可配 `~/.codex/config.toml`。

## 如何被编排器拉起

| 编排器 | 拉起方式 |
|---|---|
| **Orca** | `worktree create --name <n> --repo R --agent codex --prompt "<任务>"`；或 `terminal create --worktree active --command "codex"` |
| **CMUX** | `new-surface --type agent-session --provider codex`；或 surface 内 `codex` + `cmux send` 派单 |
| **Paseo** | `create_agent {provider: "codex/gpt-5.4", title, initialPrompt, settings: {features: {"fast_mode": true}}}` |

## 可插拔

- Codex 只是执行单元；换 Claude/OpenCode 不改变编排器侧任何配置。
- 适合前沿编码任务；fast mode 用于轻任务（对应"编排重量分级"的轻/中度）。

---

相关文件：`connectors/agents/README.md`、`connectors/agents/claude-code.md`、`connectors/agents/opencode.md`。
