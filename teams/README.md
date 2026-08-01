# 团队搭建向导（Team Setup Wizard）

> 拿到本 skill 后，如何**一步步搭一个可协作的团队**。以**交互式问答**引导，产出可执行清单。
> 默认角色 = **秘书（Secretary）**——所有开启的对话默认都是秘书角色，用户每次从秘书视角拉对话。

## 向导交互脚本（5 步）

```
Step 1 · 调度级别
  问：这次任务多"重"？轻(单任务/快问) / 中(多任务/派单验收) / 重(状态机/流水线)
  用户当前阶段：先做轻度。

Step 2 · 团队类型
  问：搭什么团队？研究团队(第一套) → 后续可扩展 执行团队 / 审查团队
  ⚠️ 第一套 = 研究团队；模板见 teams/research-team.md

Step 3 · 角色 → Agent
  问：秘书 / Manager / 研究员 各用什么 Agent？（OpenCode / Claude / Codex）
  参考下表"推荐 Agent"列。

Step 4 · 角色 → 模型
  问：各角色用什么模型？参考下表"推荐模型"列。

Step 5 · 拉起
  产出可执行清单（角色 / 运行位置 / Agent+模型 / 命令），在编排器里拉起。
  ⚠️ 团队具体起在哪（Paseo / Orca / CMUX）可**询问用户**——编排器之间可协作。
```

## 默认角色：秘书

- **所有对话默认是"秘书"角色**；用户每次从秘书视角拉起对话。
- 秘书 = 人类与团队之间的**唯一对话入口**：听需求、拆给团队、汇总回传。
- **秘书 = 人机桥梁（抹平技术 ↔ 人类）**：Manager 专心干技术，秘书把技术汇报**转译成人类可读的 HTML 报告单**交给用户（白话结论 + 决策点 + 下一步，而非 raw 技术回执）。这是秘书的**核心输出职责**，不是可选项。
- **秘书位置**：大概率在 **Paseo** 里拉起（`create_agent`），或先直接对话（轻任务不必正式建 agent）。

## 生产语言（可配置，默认全中文）

- **整个生产环节用同一语言**（默认中文）：文档交接、沟通、思维链、汇报全走它；代码标识符保持英文、注释用生产语言。
- **唯一例外**：终端派单指针必须纯 ASCII（终端 send 非 ASCII 会编码损坏）——只发文件路径，正文在文件里。
- 语言随用户偏好配置，写进团队协议第 0 条；改语言时全团队同步更新。

> ⚠️ 分工铁律：**Manager 不对用户，秘书不碰技术**。技术细节（dispatch/submission 原文、代码、日志）留在文件层；人类只读秘书转译后的报告。

## 角色-模型推荐表

| 角色 | 推荐 Agent | 推荐模型 | 说明 |
|---|---|---|---|
| 秘书 | OpenCode | deepseek-v4-flash | 对话轻快 |
| Manager | OpenCode | deepseek-v4-pro | 推理强 |
| 研究员 | OpenCode | deepseek-v4-flash | 信息收集 |

> Agent 可换（Claude/Codex 均可，见 `connectors/agents/`）；模型可按需覆盖。

## 拉起命令模板

| 角色 | 运行位置 | 命令 |
|---|---|---|
| 秘书 | **Paseo**（或先直接对话） | `create_agent {title: "秘书", provider: <映射>, initialPrompt}` / `paseo run ...` |
| Manager | Orca | `orca worktree create --repo <R> --name <n> --agent opencode --prompt "你是我团队的 Manager..."` |
| 研究员 | CMUX surfaces / Orca worktrees | `cmux new-surface --type agent-session --provider opencode` / `orca worktree create ... --agent opencode` |

> **团队位置询问**：团队具体起在哪（Orca worktree / CMUX surfaces）可问用户——秘书(Paseo) ↔ Manager ↔ 研究员(Orca/CMUX) 之间可互相协作，编排器选择不绑定职责。

## 向导输出（可执行清单示例）

```
【轻度 · 研究团队】
角色     位置        Agent     模型                命令/说明
秘书     Paseo      opencode   deepseek-v4-flash   create_agent 或直接对话
Manager  Orca       opencode   deepseek-v4-pro     worktree create --agent opencode
研究员×N CMUX       opencode   deepseek-v4-flash   new-surface --provider opencode ×N
```

## 角色 → 技能映射（去 skill-routes/ 自取）

| 角色 | 配什么技能 | 用法 |
|---|---|---|
| 秘书 | grilling / grill-me（澄清需求）+ handoff（交接给团队）+ ask-matt（路由） | 见 `skill-routes/` 对应文件 |
| Manager | to-tickets（拆工单）+ grill-with-docs（追问留档）+ domain-modeling（共享语言）+ wayfinder（大型规划） | 见 `skill-routes/` 对应文件 |
| 研究员 | **research**（核心，查证）+ domain-modeling（共享术语）+ handoff（回执传递） | 见 `skill-routes/` 对应文件 |

> 📂 **去 skill-routes/ 找技能**：Agent 被派给某角色后，先读 `skill-routes/README.md` 技能总表定位自己的技能集，需要细节再打开对应 `<技能>.md` 浓缩文件——不强制全文加载，需要时自取。

---

相关文件：`teams/research-team.md`（研究团队模板）、`skill-routes/`（技能路由）、`philosophy.md`（编排重量分级）、`management.md`（系统架构）、`connectors/agents/`（Agent 接入）、`connectors/orchestrators/`（编排器接入）。
