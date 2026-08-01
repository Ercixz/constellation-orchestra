# Orca — 编排器档案

> 分类：**编排器**（提供 worktree 隔离 + 内置任务编排的"场"）。
> 接入总表见 `connectors/agents/README.md` 与 `connectors/orchestrators/`；哲学见 `philosophy.md`。

## 编排器四问

### 1. 用什么 CLI/界面连接它

- CLI：`orca`（需 runtime open：`orca open` 启动并等 runtime 可达；多数命令前置）。
- 无头：`orca serve [--port p] [--pairing-address host] [--mobile-pairing] [--no-pairing] [--project-root p] [--recipe-json]`。
- 状态：`orca status [--json]`（app/runtime/graph 就绪度）、`orca diagnostics memory`、`orca agent-context`。
- 远程 runtime：`orca environment add --name <n> --pairing-code <code>`；或用 `ORCA_PAIRING_CODE` / `ORCA_ENVIRONMENT`。
- 前提：**本场禁 `git init`，不新建 worktree，只用 `orca worktree current` 评估当前目录。**

### 2. 如何沟通、协作、连接到一起

- 编排闭环：`orchestration run-create` → `task-create` → `dispatch` → `worker-start` → `gate-create`（决策门）→ `gate-resolve` → `worker-read` 验收（全命令语义见下方"任务系统（orchestration）语义详解"）。
- 消息：`orchestration send` / `check` / `ask`（阻塞问协调者）/ `reply` / `inbox`。
- 拉起 agent：`worktree create --name <n> --repo R --agent <id> --prompt "<任务>"`；或 `terminal create --worktree active --command "codex|claude|opencode"`。
- 与外部协作：worktree 内 terminal 可被 CMUX 面板承载；agent 干完可通过 `paseo send` 回传给 Paseo 主管（见 `connectors/agents/README.md`）。

### 3. 如何在各空间内理解空间组织结构

Orca 的空间是**三层叠加**（详见 SKILL.md 心智模型）：

| 层 | 空间单元 | 说明 |
|---|---|---|
| 抽象层 | Project / Group | 只存在于 Orca 数据库，无磁盘实体；"分组标签" |
| 真实层 | Repo（git 目录）/ Folder WS（普通目录） | 指向磁盘真实路径 |
| 显示层 | Tab / Pane / Terminal | 界面布局 |

- Repo → worktree（`repo::路径` 平级，git 隔离）；Folder WS → 直接改原目录（worktreeMeta 内嵌套 `repo::路径::workspace:<id>`）。
- Selector：`--repo` = `id:` / `name:` / `path:`；`--worktree` = `id:` / `name:` / `branch:` / `issue:` / `path:` / `active` / `current`。
- 发现用 selector，重复操作用 handle（`terminal list --json` 的 handle）。

### 4. 编排 GUI 关系（多开窗口/切分 pane/布局）

- Orca 本体是 GUI app：侧边栏 = Project/Group 树；主区 = worktree 视图；终端区 = Tab/Pane。
- 终端：`terminal split` / `switch` / `focus` / `close [--tab]` / `rename`；`terminal create [--focus]`。
- 与 CMUX 协同：Orca 的终端可并入 CMUX 布局多面板；也可以反过来在 CMUX surface 里 `orca open`。

## 命令速查（详）

### 账户 / skill

- `orca account add --agent claude|codex` / `account list`；`skills list|get|install|update`。

### Repo / Worktree（需 git）

- `orca repo add --path <p>` / `repo list` / `repo show` / `repo set-base-ref --ref <r>` / `repo search-refs --query <q>`
- `orca worktree list [--repo R] [--limit n]` / `show --worktree <sel>` / `current [--json]` / `ps [--limit n]`
- `orca worktree create --name <n> [--repo R|--project id|--project-host-setup id] [--agent id] [--prompt t] [--base-branch ref] [--issue n] [--parent-worktree sel|--no-parent] [--activate]`
- `orca worktree set --worktree <sel> [...]` / `rm --worktree <sel> [--force] [--run-hooks]`

### 文件 / 终端

- 文件：`orca file open/diff/open-changed`
- 终端：`orca terminal list|show|read|create|send|wait|stop|split|switch|focus|close|rename`

### 编排（task→dispatch→gate→worker→message）

- `orchestration run-create|run-use|run-current|run-list|run-show`
- `task-create|task-list|task-update`；`dispatch|dispatch-show`
- `send|check|ask|reply|inbox`；`gate-create|gate-resolve|gate-list`
- `worker-start|worker-show|worker-read|worker-stop|worker-abandon`
- `coordinator-start|coordinator-stop`（legacy）；`reset`

## 任务系统（orchestration）语义详解

> 实测结论：Orca 的"任务"= **agent 会话**；"协作" = 会话间的文件/消息联动。本地编排**没有独立任务看板 GUI**，靠 Agent Dashboard + Agents Feed 间接呈现。外部 issue（GitHub/Jira/Linear/GitLab）只是**任务来源**，不是本地编排任务。

### 命令族语义（按生命周期排序）

| 命令 | 语义 | 关键参数 |
|---|---|---|
| `run-create` | 创建 Run = **命名空间 + 收件箱（inbox）**。本身**不调度、不放置 worker** | `--objective` |
| `run-use` / `run-current` / `run-list` / `run-show` | 选/查当前 Run | — |
| `task-create` | 建任务，有 ID/依赖/父子 | `--spec`、`--task-title`、`--deps`、`--parent`、`--run` |
| `task-list` / `task-update` | 查/改任务 | — |
| `dispatch` | 派发给**终端**执行 | `--task <id>`、`--to <terminal>` |
| `dispatch-show` | 查看派发状态 | — |
| `worker-start` | 拉起**受监督 worker** | `--task <id>`、`--agent <agent>`、`--worktree current\|new-child` |
| `worker-show` / `worker-read` | 看 worker 状态 / 读产出 | — |
| `worker-stop` / `worker-abandon` | 停 / 弃 worker | — |
| `send` | 发消息 | — |
| `ask` | **阻塞**问协调者 | — |
| `reply` / `inbox` | 回复 / 收件箱 | — |
| `gate-create` | 建决策门：任务**阻塞**直到人类/主管决策 | `--task <id>`、`--question`、`--options` |
| `gate-resolve` / `gate-list` | 解门 / 列门 | — |

### Run vs Task vs Worker 的关系

```
Run（命名空间 + 收件箱，run-create --objective）
└── Task（ID/依赖/父子，task-create）
    ├── 派发给终端：dispatch --task <id> --to <terminal>
    └── 受监督 worker：worker-start --task <id> --agent <agent> [--worktree current|new-child]
    └── 决策门：gate-create --task <id> --question --options → gate-resolve（阻塞等决策）
```

### GUI 呈现三入口

| 入口 | 位置 | 看什么 | 操作 |
|---|---|---|---|
| **Agent Dashboard** | Settings → Experimental → Agent Dashboard（已启用 in-window + showIdle） | 看板列 Needs You / Working / Done / Idle，卡片 = agent 会话状态 | 点击卡片跳转对应终端 |
| **Agents Feed** | 侧边栏 Agents 入口 | 线程式活动流（完成/阻塞/回复预览） | `Cmd+F` 过滤 |
| **Worktree 卡片** | worktree 视图 | 行内 agent 状态点：🟡 工作 / 🟢 完成 / 🔴 阻塞 / ⚪ 空闲 | 观察状态点即知会话进展 |

**澄清**：**任务来源选择器（GitHub/Jira/Linear/GitLab）= 外部 issue 提供商**，是"从外部拉任务进来"的入口，**不是**本地编排任务；本地编排任务的呈现靠 Dashboard + Feed + worktree 卡片间接完成。

### 协作演示复现（已跑通）

3 个 worktree 各配 1 个 opencode agent，靠任务间依赖协作（alpha 产 features.md → beta 产 risks.md → gamma 读两者产 roadmap.md）：

```bash
# 1. 建 3 个 worktree
orca worktree create --repo <R> --name task-alpha
orca worktree create --repo <R> --name task-beta
orca worktree create --repo <R> --name task-gamma
# 2. 各起 agent 终端
orca terminal create --worktree task-alpha --command "opencode -m <model>"
orca terminal create --worktree task-beta  --command "opencode -m <model>"
orca terminal create --worktree task-gamma --command "opencode -m <model>"
# 3. 派任务（alpha 产 features.md、beta 产 risks.md、gamma 读两者产 roadmap.md）
orca terminal send --terminal <alpha-h> --text "产出 features.md" --enter
orca terminal send --terminal <beta-h>  --text "产出 risks.md"   --enter
orca terminal send --terminal <gamma-h> --text "读 features.md 与 risks.md，产出 roadmap.md" --enter
# 4. 验证
orca worktree ps              # live 状态（GAMMA_DONE 等回传）
orca worktree show --worktree task-gamma   # 确认产出文件
```

### Computer Use / 模拟器 / 浏览器

- Computer：`computer capabilities|permissions|list-apps|list-windows|get-app-state|click|scroll|drag|type-text|press-key|hotkey|paste-text|set-value` 等
- 模拟器：`emulator list|attach|tap|type|gesture|button|rotate|exec|kill`
- 浏览器：`tab create|list|show|switch|close`、`goto|back|forward|reload|snapshot|get|eval|wait|click|fill|type|select|hover|keypress|focus|clear|drag|upload|scroll`、`device|offline|headers|credentials|media|mouse|clipboard|dialog|storage|download|highlight|inserttext|find|screenshot`、`exec --command "..."` 透传

## 工作流

1. **隔离任务**（git 环境）：`worktree create --name <t> --agent <id> --prompt "<任务>"`；或 `terminal create --worktree active --command "codex"`。
2. **本场无 git**：只 `orca worktree current` 评估。
3. **编排闭环**：task-create → dispatch → worker-start → gate-create → gate-resolve → worker-read。
4. **浏览器流程**：tab create/goto → snapshot（拿 `@e1` ref）→ click/fill/keypress → 导航后**必须重新 snapshot**（ref 会变）。

---

相关文件：`philosophy.md`、`connectors/orchestrators/cmux.md`、`connectors/orchestrators/paseo.md`、`connectors/agents/README.md`、`SKILL.md`。
