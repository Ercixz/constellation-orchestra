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

## ⚠️ 踩坑警示：不要设全局第三方 provider

- **`~/.codex/config.toml` 被 CLI 与 ChatGPT.app（desktop）共享**，二者没有独立配置。
- 若在 config.toml 里设全局 `model_provider = "<第三方>"`（如 DeepSeek）：
  - desktop 也会被带成第三方 provider → GUI 进程拿不到该 provider 的 `env_key` 环境变量 → 报 `Missing environment variable: ...`；
  - 官方模型也被挤掉/带偏（`model_catalog_json` 是"替换"语义，不是追加）。
- **正确做法**：全局保持官方默认；要用第三方时在 CLI 显式指定：
  `DEEPSEEK_API_KEY=xxx codex exec -c 'model_provider="deepseek"' -m deepseek-v4-flash "<任务>"`
- **本场约定（2026-08-02 起）**：codex / claude code 保持纯官方；deepseek 走 OpenCode（见 `opencode.md`）。desktop 与 CLI 之间靠环境变量差别的坑，不再碰。

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
