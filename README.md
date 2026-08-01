# 星座交响乐团 (Constellation Orchestra)

> 群星各居其位，齐奏一支交响。

工作区组织方法论 skill。它不是"怎么用工具"，而是**"怎么组织一切"**——项目结构、团队分工、任务流转、工具选型。任何 agent 装上它，就明白这套设计哲学。

## 这是什么

一个 opencode skill（也可用于 Claude Code / 其他支持 SKILL.md 的 agent），把工作区的组织方法沉淀为一份总纲 + 三份工具手册 + 一份派单协议。

| 文件 | 内容 |
|---|---|
| `SKILL.md` | 方法论总纲：心智模型 / 项目蓝图 / 团队角色 / 任务闭环 / 选型 / 踩坑 |
| `tool-cmux.md` | CMUX（终端多面板编排）命令手册 |
| `tool-orca.md` | Orca（worktree + 内置编排平台）命令手册 |
| `tool-paseo.md` | Paseo（agent 守护编排）命令手册 |
| `protocol-cmux.md` | CMUX 派单协议与踩坑记录 |

## 核心哲学（30 秒版）

- **Project 是抽象容器**，没有磁盘实体；真实的是 git Repo（worktree 隔离）和 Folder WS（直改）
- **代码用 worktree 隔离，资产/文档用 folder workspace 直改**——一个项目可以既有代码又有资产
- **团队三角色**：Manager（指挥）/ Worker（乐手）/ Reviewer（审谱）
- **任务闭环**：建单 → 派发 → 执行 → 回执 → 独立验收 → 归档

## 安装

```bash
# opencode
git clone https://github.com/Ercixz/constellation-orchestra.git ~/.config/opencode/skills/constellation-orchestra

# Claude Code（或任何读 ~/.claude/skills 的 agent）
git clone https://github.com/Ercixz/constellation-orchestra.git ~/.claude/skills/constellation-orchestra
```

安装后 agent 启动时自动加载；遇到组织架构/团队/任务流转相关问题时，会按 description 自动调用。

## 更新

改 `~/.config/opencode/skills/constellation-orchestra/`（或直接克隆这份仓库修改），然后推送：

```bash
cd <repo>
git add -A && git commit -m "更新" && git push
```

也可以直接用自然语言告诉你的 agent"更新星座交响乐团：……"，agent 会完成修改和推送。

## 许可

公开仓库，随意使用/改造/分享。
