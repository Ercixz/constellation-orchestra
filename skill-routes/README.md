# 技能路由（Skill Routes）— 团队技能索引

> 本目录是**团队技能路由**：Agent 按角色需求来这里**自取**技能说明。
> 每个文件是**浓缩版**（30-60 行，够用不占 token）；完整版见文末 mattpocock/skills 原文链接。
> 想懂这些技能的来源与全貌 → 见 `/tmp/mattpocock-research/skill-survey.md` 调研笔记。

## 技能总表

| 技能 | 分类 | 给谁用 | 何时用 | 原文（mattpocock/skills） |
|---|---|---|---|---|
| **research** | AFK | 研究员（核心） | 按工单查证一手来源，产出带引用回执 | `engineering/research` |
| **grilling** | HITL | 秘书 / Manager | 人机对话澄清需求/决策（一次一问） | `productivity/grilling` |
| **grill-me** | HITL | 秘书 | 无代码库/无状态的快速追问 | `productivity/grill-me` |
| **domain-modeling** | 词汇层 | Manager | 建/磨共享语言与术语、记 ADR | `engineering/domain-modeling` |
| **handoff** | 全员 | 全员 | 会话间/角色间交接上下文 | `productivity/handoff` |
| **to-tickets** | 产出 | Manager | 拆工单（blocking 边 + 可自证切片） | `engineering/to-tickets` |
| **grill-with-docs** | HITL | Manager | 追问 + 顺带留 CONTEXT.md/ADR | `engineering/grill-with-docs` |
| **ask-matt** | 路由 | 全员 | 不确定下一步该用哪个技能 | `engineering/ask-matt` |

> **分类说明**：AFK = agent 可独自干（可并行）；HITL = 须人机对话（agent 不自问自答）；词汇层/路由 = 底层支撑。

## 按角色取用

- **秘书**：`grilling` / `grill-me`（澄清需求）+ `handoff`（交接给团队）+ `ask-matt`（路由）
- **Manager**：`to-tickets`（拆工单）+ `grill-with-docs`（追问留档）+ `domain-modeling`（共享语言）+ `wayfinder`（大型规划，见 philosophy/management）
- **研究员**：`research`（核心，查证）+ `domain-modeling`（共享术语）+ `handoff`（回执传递）

## 用法

1. Agent 被派给某角色后，先读本索引定位自己的技能集。
2. 需要细节 → 打开对应 `<技能>.md` 浓缩文件。
3. 需要完整原文 → 点表格里的 mattpocock/skills 链接。

---

相关文件：`teams/README.md`（团队向导）、`teams/research-team.md`（研究团队模板）、`philosophy.md`（编排哲学）。
