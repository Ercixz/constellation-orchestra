# ask-matt（技能路由导航）

## 定位

> 一句话：**路由**——问"当前情况该用哪个 skill/流程"，是主流程 idea→ship 的导航器。

## 给谁用

- **全员**（秘书/Manager/研究员不确定下一步时）。

## 何时用

- 拿不准当前场景该调哪个技能。
- 想走完整工作流（从想法到落地）时定位自己在哪一步。

## 主流程速览（routing）

```
grill-with-docs（磨想法，有代码库/留档）
  ├─ (无代码库→grill-me)
  → to-spec（对话→spec）
  → to-tickets（spec→工单，带 blocking 边）
  → implement（每工单：tdd 驱动 + code-review 后提交）

wayfinder（大块模糊规划）→ 地图清后 handoff 到 to-spec → 主线
research（后台查证）→ 产出喂给 grill-with-docs 思考
handoff / compact → 会话间传递（handoff 开新会话带文件；compact 留同会话精简）
```

## 关键纪律

1. 核心流程保持**一个不中断的上下文窗口**（到 to-tickets 前别 compact/clear）。
2. 接近 smart zone 边界 → `handoff` 换新线程，别硬推。
3. 新到仓库先跑 `/setup-matt-pocock-skills` 配置一次。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/ask-matt
