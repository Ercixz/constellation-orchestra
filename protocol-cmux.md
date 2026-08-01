---
name: cmux-orchestration
description: 用 CMUX 多面板编排多个 opencode agent 协作完成任务（派单-执行-回执-验收-回传协议）。Use when you need to delegate work to multiple agents in parallel, dispatch tasks to CMUX surfaces, check worker status, or manage the Stage-D-NPM-Cutover/Cluster workspaces.
---

# CMUX 多 Agent 编排协作

用 CMUX 终端复用器的多个面板（surface）并行跑 opencode agent 干活，Manager（你）通过 CLI 派单/验收。

## 基础设施

- CMUX CLI: `/Applications/cmux.app/Contents/MacOS/cmux`（PATH 里无 cmux，用全路径）
- 编排 workspace: `workspace:8` = "Stage-D-NPM-Cutover"（UUID `1F5D2690-75F2-4459-AF46-480EDE0F5310`），6 surface 平铺 3列×2行，各 76 列
- 6 个 surface：32/34/36/37/39/40（UUID 每次查询 `cmux tree --id-format both`）
- 每个 surface 跑: `opencode -m deepseek/deepseek-v4-flash`（system 主题，TUI）
- 任务单: `/Users/lueric/Isonoimic/orchestrator-test/tickets/XXX.md`
- 回执: `/Users/lueric/Isonoimic/orchestrator-test/receipts/XXX.json`
- 工作目录: `/Users/lueric/Isonoimic/Endpoint-Cluster-Console`

## 核心命令速查

```bash
CMUX=/Applications/cmux.app/Contents/MacOS/cmux
W=1F5D2690-75F2-4459-AF46-480EDE0F5310

$CMUX tree --workspace $W --id-format both     # 看布局 + 全部 UUID（surface 数字 ref 不稳定，用 UUID）
$CMUX read-screen --surface <UUID> --workspace $W   # 读面板屏幕（tail 看底部状态）
$CMUX send --surface <UUID> --workspace $W "文本"   # 向面板输入
$CMUX send-key --surface <UUID> --workspace $W enter # 按键（enter/escape/up/down/ctrl+c）
$CMUX workspace list                              # 所有 workspace
$CMUX new-pane --workspace $W --direction right   # 新面板
$CMUX move-surface --surface <UUID> --pane <UUID> --workspace $W  # 移动 surface 到 pane
$CMUX split-off --surface <UUID> down --workspace $W  # 从 tab 拆成独立 pane
$CMUX resize-pane --pane <UUID> -L --amount 5 --workspace $W  # 调 pane 宽度
```

## 协作协议（Manager 视角）

1. **写任务单** `tickets/YYYY-MM-DD-NNN-slug.md`，必含：
   - Target lock（目标主机/允许/禁止）
   - 背景（Manager 已验证的 live 事实）
   - 执行步骤 + 回执 JSON schema
   - 完成后必须 `paseo send <manager-id> --no-wait "..."` 汇报
2. **派单**：`cmux send` 一句话（"新任务 NNN：... 任务单：<路径>。立即执行，完成写回执 + paseo send 汇报"）+ enter
3. **等回执**：worker 完成会 `paseo send` 汇报（勿频繁轮询）
4. **独立验收**：收到回执后**自己验证 live 状态**（curl/ssh/docker），不轻信回执内容
5. **归档**：任务闭环后更新 `AI-Persistence/`（Memory-Bucket + Agent-Context + prompt-for-next-agent + Task-List）

## Worker 回传（零 hook 零脚本，纯约定）

- worker 收工/遇到情况时自己执行：`paseo send f2e6aa67-7f67-4c3b-a97c-95e2ccb0b90a --no-wait "NNN 完成：一句话摘要（回执+status）"`
- paseo 路径：`~/.local/bin/paseo`（或 `/Applications/Paseo.app/Contents/Resources/bin/paseo`）
- 关键：这写在每个任务单的"完成后必须执行"段

## 踩坑记录（必读）

1. **渲染错乱根因**：opencode TUI 需 ≥60 列；窄 pane（33列）会碎屏。6 surface 必须 3列×2行平铺（各 76 列），勿横排 5+ 个
2. **QUEUED 卡死**：模型生成中又发消息 → 新消息排队。Esc 两次可打断；彻底卡死就 `/exit` 退出重开（`opencode -m deepseek/deepseek-v4-flash -s <旧sessionID>` 恢复原会话）
3. **`-s` 恢复旧 session 会覆盖命令行模型**：恢复后需在 TUI 内 `/models` 重新切 flash（列表过滤 "DeepSeek V4 Flash"，选第 3 项非 Free 版）
4. **surface 数字 ref 不稳定**：move/split 后编号会变，一律用 UUID
5. **同一 surface 勿叠加任务**：会 QUEUED；一个 surface 一次只跑一个任务
6. **模型切换交互**：`/models` → enter → 输入过滤词 → down → enter（窄屏下列表显示不全，用过滤最稳）
7. **会话恢复**：opencode session 存 `~/.local/share/opencode/opencode.db`，用 `opencode session list` 查 ID
8. **主题**：`~/.config/opencode/tui.json` 设 `"theme": "system"`（终端原生配色，无背景色块）
9. **Esc 键**：`session_interrupt` 默认 escape；"esc again to interrupt" 需按两次
10. **安全**：含凭据任务（API key/token）绝不写入回执/知识库明文；key 只进本机 600 权限配置；派单时经 CMUX 会话单独传递

## 与 Paseo 的配合

- Manager 通常本身跑在 Paseo 里（agent id 见 `~/.paseo/agents/` 或 `paseo ls`）
- worker 通过 `paseo send <manager-id>` 回传 → Manager 在 Paseo 会话自动收到
- 派任务前若 surface 卡 QUEUED：Esc×2 → 仍不行 `/exit` + 重开（保留旧 session）

## 相关文件

- 任务单/回执：`/Users/lueric/Isonoimic/orchestrator-test/`
- 派单方法论：`/Users/lueric/Isonoimic/orchestrator-test/HOW_I_DISPATCH_SUBAGENTS.md`
- 编排 workspace 布局：3列×2行（surface 39/36/37 上行，40/32/34 下行）
