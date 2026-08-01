# grill-with-docs（追问 + 留档）

## 定位

> 一句话：`grilling` + `domain-modeling` 的**薄封装**——狠问澄清的同时，把术语/决策落进 `CONTEXT.md`（glossary）+ `docs/adr/`，留下纸面痕迹。

## 给谁用

- **Manager**（澄清歧义并沉淀决策依据，供团队追溯）。

## 何时用

- 有代码库 / 有仓库要沉淀上下文时。
- 想"对话→结构化文档"，让决策可追溯（为什么选了 A）。
- （对照：无代码库 / 不想留档 → 用 `grill-me`）

## 关键纪律

1. 复用 `grilling` 原语：一次一问、事实自查、决策问人、未共识不动手。
2. 复用 `domain-modeling` 纪律：CONTEXT.md 只当 glossary；ADR 三条件才写。
3. **stateful**：学到的记进 CONTEXT.md/ADR，不散失在对话里。
4. 未确认共识前不动手。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs
