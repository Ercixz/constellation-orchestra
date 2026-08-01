# handoff（角色间交接）

## 定位

> 一句话：把当前对话压缩成**交接文档**，让另一个 agent 接续工作。

## 给谁用

- **全员**（秘书↔Manager↔研究员之间的上下文传递）。

## 何时用

- 会话将满 / 需要换上下文窗口 / 角色之间交接（研究员回执 → Manager 汇总 → 秘书呈现）。
- 跨编排器传递：Orca worktree / CMUX surface / Paseo agent 之间。

## 关键纪律

1. **存临时目录**（用户 OS 的 tmp，不污染 workspace）。
2. **含 "suggested skills" 节**——告诉接手 agent 该调用哪些 skill。
3. **引用不复制**：已有 artifact（spec/plan/ADR/issue/commit/diff）按路径或 URL 引用，不重复内容。
4. **脱敏**：API key、密码、PII 一律擦掉。
5. 有 argument 时按"下一会话要干什么"量身定制。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/productivity/handoff
