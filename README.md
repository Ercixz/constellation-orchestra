# 星座交响乐团 (Constellation Orchestra)

> 群星各居其位，齐奏一支交响。

工作区组织方法论 skill。它不是"怎么用工具"，而是**"怎么组织一切"**——项目结构、团队分工、任务流转、工具选型。任何 agent 装上它，就明白这套设计哲学。

## 核心哲学（30 秒版）

- **二元论（本 skill 的灵魂）**：整个生态只有两类东西——**编排器**（Orca/CMUX/Paseo，组织/调度/派发的"场"）与 **AI Agent**（Claude Code/Codex/OpenCode，执行任务的"单元"）。二者都是**可插拔的环节**，唯一的定位器就是这两类。
- **编排重量分级**：任务分轻/中/重——能轻度绝不重度，编排器/Agent 可插拔的意义就在于此
- **系统架构**：GitHub Issues 作任务载体 + Orca 编排 + 多 Agent 协作 + wayfinder 规划（四层架构）
- **团队向导**：默认角色=秘书（Paseo），按向导搭秘书/Manager/研究员团队（见 `teams/`）
- **Project 是抽象容器**，没有磁盘实体；真实的是 git Repo（worktree 隔离）和 Folder WS（直改）
- **代码用 worktree 隔离，资产/文档用 folder workspace 直改**
- **团队三角色**：Manager（指挥）/ Worker（乐手）/ Reviewer（审谱）
- **任务闭环**：建单 → 派发 → 执行 → 回执 → 独立验收 → 归档

## 这是什么

一个 opencode skill（也可用于 Claude Code / 其他支持 SKILL.md 的 agent），把"编排哲学 + 工具接入方法 + 团队技能路由"沉淀为一份总纲 + 一份哲学 + 一份管理 + 团队向导 + 技能路由 + Connector 两层结构 + 一份派单协议。

| 文件 | 内容 |
|---|---|
| `SKILL.md` | 总纲：二元论 / 重量分级 / 心智模型 / 项目蓝图 / 团队角色 / 任务闭环 / 导航 |
| `philosophy.md` | 【设计哲学】编排器 vs AI Agent 二元论、可插拔、主管-子 Agent、编排重量分级、团队协作分派、星座乐团 |
| `management.md` | 【管理】系统架构：GitHub 任务平台（四层架构 / 任务闭环 / 组件职责 / 跨项目 / 决策链） |
| `teams/README.md` | 团队搭建向导入口（交互 5 步 / 角色-模型推荐表 / 拉起命令模板） |
| `teams/research-team.md` | 研究团队模板（结构 / 角色职责 / 消息流 / 工单格式 / 拉起） |
| `skill-routes/README.md` | 技能路由索引（8 技能总表：research/grilling/grill-me/domain-modeling/handoff/to-tickets/grill-with-docs/ask-matt） |
| `skill-routes/<技能>.md` | 各技能浓缩版（定位 / 关键纪律 / 原文链接，30-60 行） |
| `connectors/orchestrators/orca.md` | Orca 档案（四问 + 命令速查） |
| `connectors/orchestrators/cmux.md` | CMUX 档案（四问 + 命令速查） |
| `connectors/orchestrators/paseo.md` | Paseo 档案（四问 + 命令速查） |
| `connectors/agents/README.md` | AI Agent 接入总览 + 被编排器拉起的三种方式 |
| `connectors/agents/claude-code.md` | Claude Code（供应商配置/皮肤/拉起） |
| `connectors/agents/codex.md` | Codex（认证/fast mode/皮肤/拉起） |
| `connectors/agents/opencode.md` | OpenCode（provider 配置/TUI 皮肤/拉起） |
| `protocol-cmux.md` | CMUX 派单协议与踩坑记录 |

每个 `connectors/orchestrators/*.md` 都回答"编排器四问"：CLI 连接 / 沟通协作 / 空间结构 / GUI 关系。

## 安装

```bash
# opencode
git clone https://github.com/Ercixz/constellation-orchestra.git ~/.config/opencode/skills/constellation-orchestra

# Claude Code（或任何读 ~/.claude/skills 的 agent）
git clone https://github.com/Ercixz/constellation-orchestra.git ~/.claude/skills/constellation-orchestra
```

安装后 agent 启动时自动加载；遇到编排/组织/团队/任务流转相关问题时，会按 description 自动调用。

## 更新

改 `~/.config/opencode/skills/constellation-orchestra/`（或直接克隆这份仓库修改），然后推送：

```bash
cd <repo>
git add -A && git commit -m "更新" && git push
```

也可以直接用自然语言告诉你的 agent"更新星座交响乐团：……"，agent 会完成修改和推送。

## 许可

公开仓库，随意使用/改造/分享。
