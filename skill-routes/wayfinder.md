# wayfinder（大型任务规划 · 决策票地图）

## 定位

> 一句话：把**大到一次 agent session 装不下**的模糊大任务，画成一张 **decision-ticket 决策票地图**，一次一票解决，直到路清晰。**Plan, don't do**——默认只产决策，不产交付物。

## 给谁用

- **Manager**（大型规划专用，仅限重量档大项目）。
- ⚠️ **不改变执行派发模式**：wayfinder 只负责"想清楚怎么拆"，拆完仍由 Manager 中心化派单给 Worker/Researcher。不是去中心化认领制。

## 何时用

- 大任务模糊、跨多个 session、决策链不清晰时。
- 任务不大/决策清晰 → 直接 to-tickets 拆工单即可，不升 wayfinder。

## 票型分类（按是否需要人参与）

| 票型 | 含义 | 人参与？ | 谁执行 |
|---|---|---|---|
| `research` | 查证信息，出结论 | ❌ AFK 自动 | Manager 派 Researcher |
| `task` | 直接实施 | ❌ 自动 | Manager 派 Worker |
| `prototype` | 做原型验证方案 | ✅ HITL（要人确认原型） | 上报秘书 → 秘书问用户 |
| `grilling` | 人机对齐需求/决策 | ✅ HITL（要人对话） | 上报秘书 → 秘书问用户 |

> **HITL 上报机制（本体系定制）**：需要人的票，Manager **不直接找用户**——写"上报单"（问题 + 选项 + 背景）→ 传给秘书 → 秘书与用户对话 → 秘书回传答案 → Manager 解票继续。人只和秘书交谈（层级通信铁律）。

## Local Markdown Tracker（本体系落地）

无 GitHub 时用本地文件制，地图与票为 markdown 文件：

```
agent-journal/wayfinder/<feature>/
├── map.md                 ← 地图（Destination / Notes / Decisions so far / Not yet specified / Out of scope）
└── tickets/
    ├── ticket-001-grilling-<topic>.md   ← 每票一文件：问题/类型/blocking/状态
    ├── ticket-002-research-<topic>.md
    └── ...
```

- **地图是唯一真相**（source of truth）：状态全在文件里，不依赖任何会话记忆。
- **Manager 会话可短命**：每次推进 = 拉起 → 读 map.md → 解 frontier 票 → 产物落盘 → 关会话。
- frontier = 所有 blocker 已完成、未认领的票。
- 关闭票 = 在 map.md 的 Decisions so far 加一行（用名字引用，不用裸 id）。

## 关键纪律

1. **Plan, don't do**：地图阶段只拆票、出决策，不实施。
2. **一次一票**：每个 session 只解一张 frontier 票（research 除外可批量）。
3. **每票一个决策**：票 = 一个待答问题，不是执行切片。
4. **blocking 边**：票间依赖标清，frontier 按此推进。
5. **名字引用**：人类可读处用票名，不用裸编号。
6. **fog of war**：Not yet specified 区先挂雾，frontier 前进时毕业成票。
7. **Out of scope**：判定超出目的地的，关闭永不毕业。
8. **HITL 票走秘书**：grilling/prototype 票上报秘书，不直接对用户。

## 与相关技能的关系

- **上游**：grilling / grill-with-docs（grilling 票的追问原语）、research（research 票）、prototype（prototype 票）、to-tickets（下游：票清后拆实施工单）。
- **不是任何技能的"升级版"**：wayfinder 是规划层容器，grilling/grill-with-docs 是执行层原语——层级不同，互补。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/wayfinder
