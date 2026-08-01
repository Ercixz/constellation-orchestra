# Agent：OpenCode

> 分类：**AI Agent**（执行单元）。本场主用 agent。
> 总览见 `connectors/agents/README.md`；哲学见 `philosophy.md`。

## CLI 连接

- 命令：`opencode`
- 前提：已安装（brew / npm）；已配置 provider（见下）

## 供应商配置

- **provider 配置**：`~/.config/opencode/opencode.json`（或 jsonc）里配置 provider + API key + base URL + model。
- 本场示例：`deepseek/deepseek-v4-flash`（命令行 `-m deepseek/deepseek-v4-flash`）。
- **Base URL**：自定义网关可在此配（如 LiteLLM）。
- **TUI 皮肤**：`~/.config/opencode/tui.json` 设 `"theme": "system"`（终端原生配色，无背景色块）。
- ⚠️ 凭据（API key）只进本机 600 权限配置，绝不入文档明文。

## 皮肤 / 设置

- TUI 需 **≥60 列**渲染；窄 pane（33 列）会碎屏。
- `-s <sessionID>` 恢复旧会话；恢复后模型会被覆盖，需在 TUI 内 `/models` 重切。
- skill 目录：`~/.config/opencode/skills/`（本 skill 即安装于此）。

## 如何被编排器拉起

| 编排器 | 拉起方式 |
|---|---|
| **Orca** | `worktree create --name <n> --repo R --agent opencode --prompt "<任务>"`；或 `terminal create --worktree active --command "opencode -m deepseek/deepseek-v4-flash"` |
| **CMUX** | `new-surface --type agent-session --provider opencode`；或 surface 内 `opencode -m <model>` + `cmux send --surface <UUID>` 派单 |
| **Paseo** | `create_agent {provider: <映射>, title, initialPrompt}`（provider 映射见 `~/.paseo/orchestration-preferences.json`） |

## 可插拔

- OpenCode 只是执行单元；换 Claude/Codex 不改变编排器侧任何配置。
- 本场短任务默认用它（flash 模型轻快，对应"编排重量分级"的轻度）。

---

相关文件：`connectors/agents/README.md`、`connectors/agents/claude-code.md`、`connectors/agents/codex.md`。
