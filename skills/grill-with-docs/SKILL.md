---
name: grill-with-docs
description: 狠问澄清一个计划或设计——本项目本地化版：只走 GRILL 追问原语，不自动留档文档。
disable-model-invocation: true
---

# grill-with-docs（追问澄清 · 本地化版）

> 本项目（constellation-orchestra）定制：原版是「grilling + domain-modeling 薄封装」（追问并自动写 ADR/glossary）。本版**只用 GRILL 追问**，文档留档由项目流程负责（秘书编译 GDD / Manager 按需写 ADR），本技能不自动产文档。

## 用法

运行一次 **grilling 会话**（追问原语），遵循其纪律：

- 一次只问一个问题（一次问多个会让人懵）
- 事实去环境查（filesystem/tools 能查到的），不问用户
- 决策是用户的——逐条提给用户，等回答后再继续
- 逐分支走决策树，解决决策之间的依赖
- **未达共识不动手**——确认双方理解一致才执行

## 本项目角色归属（层级通信铁律）

- **追问执行**：秘书（唯一对话入口，人只和秘书交谈）。
- **提问设计**：Manager 写上报单（问题 + 选项 + 背景）→ 秘书转述并记录（不自答）→ 回传 Manager。
- **Worker/Researcher 不参与人机对话**。
- 产出：对话记录 → 秘书编译进 GDD（`Docs/10-Knowledge/GDD-*.md` 第 8 章）或按需交 Manager 落 ADR。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs
