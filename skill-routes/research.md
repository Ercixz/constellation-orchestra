# research（研究员核心技能）

## 定位

> 一句话：对高信任**一手来源**查证一个问题，产出**带引用的 Markdown 回执**，跑为**后台 agent**。

## 给谁用

- **研究员**（核心工具）。Manager 也可用于自己先查证再追问。

## 何时用

- 需要"文档/API/事实"类信息，且当前工作目录之外有知识要查。
- 想并行多线程调研（每个 research 是一个后台 agent，可同时跑多个）。
- wayfinder 的 `research` 决策票正是用本技能并行解决。

## 关键纪律

1. **只信一手来源**：官方文档、源码、spec、first-party API——不采信二手转述；每条声明追回到拥有它的源头。
2. **产出单文件 Markdown**：逐条标注来源引用。
3. **放仓库既有惯例位置**：没有惯例就放合理处并说明。
4. **后台并行**：spin up 一个 background agent，主会话继续干活不等待。
5. research 是"喂给思考"，不替代决策——产出进主流程（grill-with-docs）做澄清输入。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/research
