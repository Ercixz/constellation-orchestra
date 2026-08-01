---
name: constellation-orchestra
description: 编排哲学与工作区组织方法论总纲（星座=组织架构 / 乐团=协同流程）。核心二元论：编排器（Orca/CMUX/Paseo）vs AI Agent（Claude/Codex/OpenCode），一切皆可插拔环节；编排重量分级（轻/中/重）+ 团队协作分派（轻/中型直连协议，不套 LangGraph）+ 系统架构（GitHub 任务平台四层架构）。Use when you need to understand or configure the orchestration philosophy, weigh a task (light/medium/heavy), dispatch a light/medium task via direct protocol (Paseo supervisor ↔ orchestrator ↔ sub-agent), understand the GitHub-Issues-as-task-platform architecture (four layers / task loop / cross-project), connect to an orchestrator or an AI agent, organize a project structure (code/assets/docs layering), set up an agent team (Manager/Worker/Reviewer), design task workflow, decide between git worktree vs folder workspace, or dispatch/verify multi-agent work. 含 connectors/orchestrators/（编排器四问详解）+ connectors/agents/（Agent 供应商/皮肤/拉起）+ protocol-cmux.md（派单协议与轻/中型任务协议速查）+ philosophy.md（含系统架构）。
---

# 星座交响乐团 — 编排哲学与工作区组织方法论

> 世界观：每个 agent 是一颗星，群星各居其位（星座=组织架构）；
> 指挥家执棒，众乐手齐奏（乐团=协同流程）。两者合一 = 本工作区的作战总纲。
>
> **本 skill 的灵魂（必读）**：编排器 vs AI Agent 的**二元论**——二者是可插拔的两个环节。详见 `philosophy.md`。

## 零、二元论：编排器 vs AI Agent（先看这个）

整个工具生态里**只有两类东西**，这是唯一的定位器：

| 类 | 成员 | 本质 |
|---|---|---|
| **编排器 Orchestrator** | Orca、CMUX、Paseo | 组织/调度/派发的"场"（空间、面板、生命周期、协议） |
| **AI Agent** | Claude Code、Codex、OpenCode | 执行具体任务的"单元"（读提示、动手、产出） |

- **三者都是编排器，只是形态不同**：Orca=worktree+内置编排，CMUX=终端多面板并行，Paseo=agent 生命周期守护。
- **三者都是子 Agent，可随时更换**：任一编排器都能拉起任一 agent。
- **唯一的定位器 = "编排" 还是 "AI Agent"**。提到任何工具先问这一句。
- 未来架构：**Paseo 主管 Agent ↔ 用户对话，主管把任务分发到 Orca 里的子 Agent**。
- 完整哲学 → `philosophy.md`。

## 一、核心心智模型（工作区组织）

### 1. Project 是抽象容器

**Project 没有磁盘实体**——它只是 Orca 侧边栏里的一个"分组标签"，把相关的真实目录组织在一起。磁盘上看到的只是普通文件夹；Orca 里它们可以映射成任意结构。

```
Project（虚拟容器，无磁盘实体）
├── Group（虚拟分类）
│   ├── Repo（真实 git 目录）→ worktree（任务副本，隔离）
│   └── Folder WS（真实普通目录）→ 直接干活（不隔离）
```

### 2. 三层叠加

| 层 | 是什么 | 例子 |
|---|---|---|
| 抽象层 | Project / Group（只存在于 Orca 数据库） | "游戏项目" |
| 真实层 | Repo / Folder WS（指向磁盘真实路径） | `/code` `/assets` |
| 显示层 | Tab / Pane / Terminal（界面布局） | 4 列 split 终端 |

### 3. git vs folder 的本质区别

| | git Repo | Folder WS |
|---|---|---|
| 前提 | 有 `.git` | 无 git，普通目录 |
| 隔离 | ✅ worktree 独立分支+目录 | ❌ 直接操作原目录 |
| 并行 | ✅ 多 agent 各改各的 | ⚠️ 会互相踩 |
| 用途 | 代码 | 资产/文档/配置 |

## 二、项目组织蓝图（代码+资产+文档分层）

**原则：代码用 worktree 隔离，资产/文档用 folder workspace 直改。**

```
MyGame/（磁盘普通目录）
├── code/    → git init（本地即可，不推送）→ Orca 注册为 Repo → worktree 多 agent 并行
├── assets/  → Orca 里 Folder WS（UI 打开）→ 单 agent 直改
└── docs/    → Orca 里 Folder WS → 单 agent 直改
```

- worktree **只针对代码文件夹**；资产文件夹**不被 worktree 复制**；侧边栏用 Group 分组。
- 代码注册：`orca repo add --path <code路径>`（需已 git init）；资产/文档：只能 UI 创建 folder workspace。
- folder → git 转换：git init 后**重新注册**（删旧添新，10 秒），会丢 folder 会话记录。

## 三、团队角色（星座的星位）

| 角色 | 职责 | 工具映射 |
|---|---|---|
| **Manager（指挥家）** | 建任务单、派发、独立验收、归档 | 主 surface / Paseo agent |
| **Worker（乐手）** | 读任务单执行、写回执、汇报 | CMUX surface / Orca worktree terminal |
| **Reviewer（审谱者）** | 独立复核回执与 live 状态，不轻信 | 空闲 surface / 独立 agent |

## 四、任务流转闭环（乐章的起承转合）

```
1. 建任务单  tickets/NNN-slug.md（Target lock + 步骤 + 回执 schema + 汇报要求）
2. 派发      cmux send 一句话（"新任务 NNN：任务单在 <路径>"）
3. 执行      Worker 读单干活，写回执 receipts/NNN.json
4. 回传      Worker 执行 paseo send <manager-id> --no-wait "NNN 完成：摘要"
5. 验收      Manager 独立复核 live 状态（curl/ssh/docker），不轻信回执
6. 归档      更新 AI-Persistence/（Memory-Bucket + Agent-Context + Task-List）
```

**铁律**：
- 生产改动需任务单 + 回执；Manager 独立验证
- 每个任务单必含：Target lock / 背景 / 步骤 / 回执 schema / paseo send 汇报
- 一个 surface 一次只跑一个任务（叠加会 QUEUED 卡死）
- 凭据绝不写入回执/知识库明文

## 五、文件导航（本 skill 结构）

```
constellation-orchestra/
├── README.md                安装/更新说明（点明二元论 + 重量分级）
├── SKILL.md                本总纲（二元论 + 重量分级 + 工作区方法论 + 导航）
├── philosophy.md           【设计哲学】二元论 / 可插拔 / 主管-子 Agent / 重量分级 / 系统架构(GitHub 任务平台) / 星座乐团
├── connectors/
│   ├── orchestrators/        编排器档案（四问详解）
│   │   ├── orca.md           Orca 档案（CLI/协作/空间/GUI 四问 + 命令速查）
│   │   ├── cmux.md           CMUX 档案（四问详解 + 命令速查）
│   │   └── paseo.md          Paseo 档案（四问详解 + 命令速查）
│   └── agents/               AI Agent 接入（供应商配置/皮肤/拉起）
│       ├── README.md         接入总览 + 被编排器拉起的三种方式
│       ├── claude-code.md    Claude Code（API key/base URL/皮肤/拉起）
│       ├── codex.md          Codex（认证/fast mode/皮肤/拉起）
│       └── opencode.md       OpenCode（provider 配置/TUI 皮肤/拉起）
└── protocol-cmux.md        CMUX 派单协议与踩坑
```

**阅读路径**：
- 想懂哲学 → `philosophy.md`（含编排重量分级 + 系统架构：GitHub 任务平台）
- 想连编排器 → `connectors/orchestrators/*.md`（每文件回答"四问"）
- 想配/拉 agent → `connectors/agents/*.md`
- 想派单协作 → `protocol-cmux.md`
- 想懂系统架构 → `philosophy.md` 第七节（四层架构/任务闭环/决策链，完整设计稿见 `AI-Persistence/Memory-Bucket/2026-08-01-System-Architecture-GitHub-Task-Platform.md`）

**编排器四问**（每个 connectors/orchestrators/*.md 都回答）：
1. 用什么 CLI/界面连接它
2. 如何沟通、协作、连接到一起
3. 如何在各空间内理解空间组织结构
4. 编排 GUI 关系（多开窗口/切分 pane/布局）

## 六、工具选型速查（乐团用什么乐器）

| 工具 | 类别 | 管什么 | 何时用 |
|---|---|---|---|
| **CMUX** | 编排器 | 终端面板编排 | 快速并行派单（运维/多任务） |
| **Orca** | 编排器 | worktree + 内置编排 | 需要隔离的开发任务、内置任务系统 |
| **Paseo** | 编排器 | agent 生命周期守护 | 长期任务、定时/心跳、跨会话 |
| **Claude Code / Codex / OpenCode** | AI Agent | 具体执行 | 被任一编排器拉起干活 |

- 编排器接入：`connectors/orchestrators/`；Agent 接入：`connectors/agents/`
- **任务重量分级（轻/中/重）**：见 `philosophy.md` 第五节——能轻度绝不重度
- **团队协作分派（轻重分离）**：轻/中型走直连协议（Paseo 主管 ↔ 编排器 ↔ 子 Agent，不套 LangGraph），见 `philosophy.md` 第六节；协议速查见 `protocol-cmux.md`
- **系统架构（GitHub 任务平台）**：四层架构 + 任务闭环 + 决策链（Gitea→GitLab→GitHub），见 `philosophy.md` 第七节
- CMUX 派单协议与踩坑：`protocol-cmux.md`

## 七、踩坑记录（指挥家备忘录）

1. **Orca 的 folder workspace 只能 UI 创建**——CLI 无命令（`repo add` 拒绝非 git）
2. **ProjectGroup（"来自项目的新组"）= 分组容器**，folder workspace 挂它下面（parentPath）
3. **folder workspace 在 worktreeMeta 里是嵌套**（`repo::路径::workspace:<id>`），git worktree 是平级（`repo::独立路径`）
4. **Orca worktree 必须 git**；本场 Endpoint-Cluster-Console 禁 git init → 只能当 folder
5. **opencode TUI 需 ≥60 列**渲染；窄 pane（33列）碎屏；6 surface 用 3×2 平铺各 76 列
6. **同 surface 勿叠加任务**；QUEUED 卡死按 Esc×2，再不行 /exit 重开（-s 恢复会话）
7. **-s 恢复旧 session 会覆盖模型**，需在 TUI 内 /models 重新切 flash
8. **含凭据任务**：key 只进本机 600 权限配置，派单经 CMUX 会话单独传递
9. **编排器是环节不是终点**：换编排器不影响 agent 层，换 agent 不影响编排层（见 philosophy.md）

## 八、相关资源

- 任务单/回执：`/Users/lueric/Isonoimic/orchestrator-test/`
- 知识库：`AI-Persistence/`（归档铁律见 AGENTS.md）
