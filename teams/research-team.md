# 研究团队模板（Research Team）

> 第一套团队模板。适用：**需要信息收集、文档梳理、多线程调研**的任务。
> 团队结构：人类 → 秘书 → Manager → 研究员×N。调度级别通常是**轻度/中度**。

## 团队结构图

```
人类（User）
   │  对话（唯一入口）
   ▼
秘书（Secretary）—— 默认角色，人类与团队的对话桥梁
   │  工单（拆解/派发）
   ▼
Manager（技术中枢）
   │  按工单派发给研究员
   ▼
研究员×N（Researcher 1..N）
   │  结构化回执
   ▼
（汇总回传）Manager → 秘书 → 人类
```

## 角色职责（含铁律）

| 角色 | 职责 | 铁律 |
|---|---|---|
| **秘书（Secretary）** | 与人类对话、理解需求、拆工单、汇总结果、回传人类 | **不碰技术**：不亲自调研/写代码，只做对话与调度 |
| **Manager** | 技术中枢：拆解工单、派发、汇总各研究员产出、形成结论 | **不对用户**：不直接与人类对话，只与秘书/研究员协作 |
| **研究员（Researcher）** | 按工单收集信息，产出**结构化回执** | **不越界**：只回答自己工单的问题，不扩大范围/不自作主张 |

## 各角色技能路由（配什么技能）

> 技能说明 → `skill-routes/` 对应文件；**Agent 自己去 skill-routes/ 自取**，不强制全文加载。

| 角色 | 配什么技能 | 用法见 |
|---|---|---|
| 秘书 | grilling / grill-me（澄清需求）、handoff（交接）、ask-matt（路由） | `skill-routes/grilling.md`、`skill-routes/grill-me.md`、`skill-routes/handoff.md`、`skill-routes/ask-matt.md` |
| Manager | to-tickets（拆工单）、grill-with-docs（追问留档）、domain-modeling（共享语言）、wayfinder（大型规划） | `skill-routes/to-tickets.md`、`skill-routes/grill-with-docs.md`、`skill-routes/domain-modeling.md` |
| 研究员 | **research**（核心查证）、domain-modeling（共享术语）、handoff（回执传递） | `skill-routes/research.md`、`skill-routes/domain-modeling.md`、`skill-routes/handoff.md` |

## 消息流闭环

```
人类 → 秘书：给需求
秘书 → Manager：拆成工单（背景/问题/范围/产出格式/时限/回执）
Manager → 研究员×N：派发工单
研究员×N → Manager：结构化回执
Manager → 秘书：汇总 + 结论
秘书 → 人类：结果呈现
```

## 任务工单格式

每张工单（给研究员）必含：

```markdown
## 背景
<为什么做这个调研 / 上下文>

## 问题
<要回答的具体问题，一个工单一个问题>

## 范围
<只查什么；明确不查什么>

## 产出格式
<结构化回执模板：结论 / 证据 / 来源 / 注意事项>

## 时限
<期望完成时间，或空>

## 回执
<产出物路径或直接粘贴>
```

## 拉起方式

| 角色 | 位置 | 命令 | 提示词要点 |
|---|---|---|---|
| 秘书 | Paseo | `create_agent {title:"秘书", provider:<映射>, initialPrompt:"你是团队的秘书，只负责与用户对话、拆工单、汇总，不亲自调研"}` | 强调"不碰技术" |
| Manager | Orca | `orca worktree create --repo <R> --name team-mgr --agent opencode --prompt "你是团队 Manager，拆解工单、派发、汇总，不直接对用户"` | 强调"不对用户" |
| 研究员 | CMUX / Orca | `cmux new-surface --type agent-session --provider opencode`（×N）；或 `orca worktree create --name researcher-N --agent opencode` | 强调"只答工单问题，结构化回执" |

> **拉起位置可询问用户**：秘书在 Paseo、Manager/研究员在 Orca 或 CMUX——具体编排器选择问用户，职责不绑定编排器（见 `connectors/`）。

## 变体扩展（后续）

- **执行团队**：研究员 → 执行者（改代码/产出物），Manager 拆的是实施工单。
- **审查团队**：Reviewer 独立复核回执/成果，不轻信（对应 SKILL.md 团队三角色）。

---

相关文件：`teams/README.md`（向导入口）、`skill-routes/`（技能路由）、`philosophy.md`（团队三角色/编排重量分级）、`management.md`（系统架构）、`protocol-cmux.md`（派单协议）。
