# 编排哲学（Design Philosophy）

> 本文件是 constellation-orchestra 的设计灵魂。所有具体工具说明（connectors/orchestrators/、connectors/agents/）都从这里长出来。
> 读任何工具文件之前，先理解这里的二元论。

## 一、二元论：编排器 vs AI Agent

整个工具生态里只有**两类东西**，这是唯一的定位器：

| 类 | 是什么 | 成员 | 本质 |
|---|---|---|---|
| **编排器 Orchestrator** | 组织、调度、派发、守护工作的**环节** | Orca、CMUX、Paseo | 是"场"：空间、面板、生命周期、派单协议 |
| **AI Agent** | 真正**执行**具体任务的**单元** | Claude Code、Codex、OpenCode | 是"演奏者"：读懂提示、动手改代码、产出结果 |

二者不是"谁比谁高级"，而是**分工不同**：
- **编排器管"怎么组织工作"**——任务单从哪来、派给谁、怎么等结果、怎么验收归档。
- **AI Agent 管"怎么干好一件事"**——具体命令、具体文件、具体推理。

**这就是本 skill 最想说明的一点**：提到任何工具时，先问"它是编排器还是 AI Agent？"。答案决定它在这套体系里扮演的角色。

## 二、可插拔性：一切皆环节

编排器和 AI Agent **都是可插拔的环节**，不是终点、不是信仰、不是绑定关系。

### 三个编排器可随时更换

Orca / CMUX / Paseo 形态不同，但**都只是编排器**：

- **Orca**：重 GUI、worktree 隔离、内置任务编排，适合"需要隔离的开发任务"。
- **CMUX**：终端复用器，多面板并行，适合"快速并行派单"。
- **Paseo**：agent 生命周期守护、定时/心跳、跨会话，适合"长期任务"。

某天用不顺手、或出现更优的编排器，直接换——**不影响 AI Agent 那一层**，因为两边通过标准连接方式（见 connectors/）解耦。

### 三个 AI Agent 可随时更换

Claude Code / Codex / OpenCode **都只是执行单元**：

- 任一 agent 都能被任一编排器拉起（Orca `worktree create --agent`、CMUX surface、Paseo `create_agent`）。
- Agent 的具体型号/品牌不重要，重要的是它"听编排器的话、干好被派的活"。
- 换 agent 不换编排器，反之亦然——**编排器与 agent 之间没有绑定**。

> 一句话：**编排器提供场，Agent 演奏曲；场可换，演奏者也可换。**

## 三、目标架构：主管 Agent → 子 Agent

未来的主架构是**主管-子 Agent**：

```
用户
 │  对话
 ▼
Paseo 主管 Agent（理解意图、拆任务、验收汇总）
 │  派发任务（create_agent / send_agent_prompt）
 ▼
Orca 子 Agent（每个子任务一个 worktree，隔离执行）
 │  写回执 / paseo send 回传
 ▼
Paseo 主管汇总 → 验收 → 反馈用户
```

- **主管（Supervisor）**：在 Paseo 里与用户对话的角色。它负责"听清要什么、拆成哪些任务、派给谁、怎么验收、怎么归档"。
- **子 Agent（Worker）**：在 Orca 里干活的角色。一个子任务 = 一个 worktree（隔离），跑一个 agent。
- 回传机制：子 Agent 干完通过 `paseo send <supervisor-id> --no-wait "NNN 完成：摘要"` 汇报，主管自动收到。

这套架构的本质仍是二元论：**主管是"编排器思维"的 agent 化身，子 Agent 是"执行"的单元**。

## 四、世界观：星座排布 + 乐团协奏

### 星座（组织架构）——每个 agent 是一颗星

- 每颗星（agent）各居其位，有自己的轨道（worktree/workspace），互不干扰。
- 星位（角色）预先排好：Manager / Worker / Reviewer。
- 星座的价值：**结构清晰、位置稳定、可预测**。

### 乐团（协同流程）——指挥家执棒，众乐手齐奏

- 指挥家（Manager/主管）执棒：起手（建任务单）、指哪（派单）、停拍（验收）。
- 乐手（Worker）齐奏：各读各的谱（任务单），各奏各的段（执行），奏完归位（回执）。
- 乐团的本质：**流程统一、节奏同步、协作有序**。

**两者合一**：星座决定"谁在哪"，乐团决定"怎么一起干活"。

## 五、编排重量分级（轻 / 中 / 重）

任务流要能**轻/重切换**：轻度任务不要重型编排（如 LangGraph/Pydantic AI 那类流程编排太重）。按任务复杂度先分级，再选编排方式：

| 级别 | 适用 | 编排方式 | 例子 |
|---|---|---|---|
| **轻度** | 单任务、快速问答、小改 | 无/极少编排，直接一个 agent 干 | CMUX 单 surface、直接对话 |
| **中度** | 多任务并行、需要派单/验收 | 文件制协议（tickets/receipts）+ 简单派单 | CMUX 多 surface、Paseo send |
| **重度** | 复杂流程、状态机、多阶段流水线 | 流程编排框架 | LangGraph / Pydantic AI |

**核心原则**：**任务多轻就多轻，能轻度绝不重度**——编排器/Agent 可插拔的意义就在于此：轻任务直接一个 agent 干，重任务才上流程编排框架，中间用文件制协议兜底。

- 判据：几个任务？要不要并行？要不要验收？要不要状态机/分支？
  - 单件、不验收 → 轻度
  - 多件、要派单验收 → 中度
  - 状态机/多阶段流水线 → 重度
- 编排器选型不绑定重量级：CMUX 可以轻（单 surface）也可以中（多 surface）；Paseo 管生命周期；Orca 管隔离 worktree。**重量分级管"怎么组织任务"，不限制用哪个编排器。**

## 六、团队协作分派：轻重分离

任务按**重量**分派到**不同的执行路径**——轻/中型走直连协议，重型才走流程编排框架。

### 三档分派表

| 档位 | 执行方式 | 状态 |
|---|---|---|
| **重型任务** | 利用 LangGraph / Pydantic AI 等**流程编排框架**（重框架：状态机、多阶段流水线） | 🔧 研发中（暂不展开） |
| **中型任务** | **文件制协议（tickets/receipts）+ 直连**：编排器互连 + 派单/回传/任务理解 | ✅ 现在要落地的 |
| **轻型任务** | **直接一个 agent 干**：编排器里创建 Agent，与 Paseo 主管直连，不套任何框架 | ✅ 现在要落地的 |

> 轻型与中型都归入**轻/中型直连协议**路径——中间**不套 LangGraph 等重框架**。重型才用流程编排框架（研发中）。

### 轻/中型任务执行路径图

```
用户 ↔ Paseo 主管 Agent（人类对话入口）
        │  （协议：paseo send / create_agent / send_agent_prompt）
        ▼
   编排器（Orca / CMUX）
        │  （协议：terminal send/read/wait、cmux send/send-key、worktree create --agent）
        ▼
   子 Agent（Claude / Codex / OpenCode）执行
        │  （回传：paseo send <主管id> --no-wait "完成：摘要"）
        ▼
   Paseo 主管 Agent 收到 → 汇报人类
```

### 关键点

- **主管 Agent 在 Paseo**（与人类对话入口）；**子 Agent 在编排器**（Orca/CMUX）里创建。
- 两者之间**直接打通**（协议层），**不经过 LangGraph 等重框架**。
- "打通" = **协议互操作**（三条）：
  1. **派单**（编排器方向）：主管 → 编排器 → 创建 agent / 发任务（`create_agent`、`send_agent_prompt`、`terminal send`、`cmux send`、`worktree create --agent`）
  2. **回传**（Paseo 方向）：子 Agent → 主管（`paseo send <主管id> --no-wait "NNN 完成：摘要"`）
  3. **任务理解**（双方共读）：tickets/receipts 文件约定——双方都能读懂任务单与回执 schema
- 本质：**让编排器（Orca/CMUX/Paseo）互相连接，通过协议方法传输任务/理解任务**，而不是再造一层重框架。

协议速查见 `protocol-cmux.md` 的"轻/中型任务协议速查"节。

## 七、系统架构：GitHub 任务平台

本场（Endpoint-Cluster-Console 及其未来所有项目）的目标架构：**GitHub Issues 作任务载体 + Orca 编排 + 多 Agent 协作 + wayfinder 规划**。完整设计稿见
`AI-Persistence/Memory-Bucket/2026-08-01-System-Architecture-GitHub-Task-Platform.md`——本文是方法论摘要，细节在知识库。

### 四层架构

```
第 1 层：任务层（GitHub Issues）     map（wayfinder 决策地图）/ 决策票 / 普通任务 issue
第 2 层：编排层（Orca/CMUX/Paseo）   Orca=worktree+任务来源选择器；CMUX=多 surface 并行派单；Paseo=Manager 主管守护
第 3 层：执行层（AI Agent）          Claude/Codex/OpenCode 在 worktree 里执行，git+API 与 GitHub 同步
第 4 层：介质层（git/GitHub API/文件制） 代码同步 + issues/PR 操作 + tickets/receipts 轻量兼容
```

### 任务流转闭环

- **规划**：用户给模糊需求 → Manager(Paseo) 调 wayfinder 建图 → GitHub 建 map + 决策票 → 逐票解析（research 并行 / grilling 人机）→ 路线图清晰。
- **执行**：Manager 在 Orca 任务来源选择器看到 issues → 点 issue 一键启动 worktree（带链接上下文）→ Worker agent 执行 → 提交 PR → GitHub 关 issue → Reviewer 复核归档。
- **轻量**：小改动/快问不建图，直接 tickets 文件制 + CMUX 派单 + Paseo send 回传。

### 组件职责表

| 组件 | 职责 | 可插拔 |
|---|---|---|
| GitHub Issues | 任务载体（map/票/普通任务），跨项目 | 可换 GitLab/Linear（代价：Orca 集成深度） |
| Orca | worktree 隔离、任务来源选择器、评审提供商 | 可换 |
| CMUX | 多 surface 并行派单（轻量档主力） | 可换 |
| Paseo | Manager 主管 Agent、生命周期守护、回传通道 | 可换 |
| wayfinder | 大型任务规划（决策地图） | 规划 skill |
| Claude/Codex/OpenCode | 执行单元 | 可换 |

### 跨项目组织

```
GitHub 组织：<org>（如 isonoimic）
├── 仓库：my-game（代码+LFS）/ endpoint-cluster（运维）/ orchestrator-test（工具链）
└── 每个仓库的 issues = 该项目的任务
Orca 侧：每仓库注册为一个 Project → 任务来源选择器统一显示所有 issues → 按项目/仓库过滤
```

### 关键决策链（防未来再纠结）

**Gitea → GitLab → GitHub**：

1. **Gitea**：自托管、轻（~124MiB），但 **Orca 任务/评审集成浅**（只读 git remote 检测，无 issue 原生任务接入）→ 弃。
2. **GitLab CE**：Orca 集成好，但 **8-16GB 内存太重**，本场 .100 只有 8GB → 弃。
3. **GitHub**：**Orca 一等公民**（任务来源/评审双集成）、已有账号、零自托管成本、wayfinder 原生支持 → **选定**。

> 取舍底线：任务载体要能**进 Orca 任务来源选择器**；太重不选；自托管优先但前提是不牺牲 Orca 集成。
> ⚠️ 曾部署 Gitea（094/095 为验证 Orca 接入，已跑通）——按本架构实施时 Gitea 退役（停容器/删路由/归档数据）。

### 与本章节的衔接

- 四层 = 二元论的具体化（任务层+介质层是"内容"，编排层+执行层是"环节"）。
- 轻/中型直连协议（本章六）是这套架构的轻量档；重型（wayfinder 规划 + 流程框架）是重量档前置。
- 介质层保留 tickets/receipts 文件制——轻量任务不强制上 GitHub。

## 八、实践推论

1. 遇到"组织/调度"问题 → 想在编排器层解决（哪个编排器、怎么派、怎么等结果）。
2. 遇到"干一件事"问题 → 想 AI Agent 层（哪个 agent、什么提示、怎么验收）。
3. **先分级再编排**：任务多轻就多轻，能轻度绝不重度（见"编排重量分级"）。
4. **轻/中型走直连，重型才上框架**：轻中型任务在编排器里直接建 agent + Paseo 主管直连（协议互操作），不套 LangGraph（见"团队协作分派：轻重分离"）。
5. 不要因为熟悉某工具就把它当成唯一解——**它是环节，可换**。
6. 新工具接入时，先分类（编排器 or Agent），再找对应 connector，四问答齐即可上手。

---

相关文件：`SKILL.md`（总纲）、`connectors/orchestrators/*.md`（编排器四问详解）、`connectors/agents/*.md`（Agent 接入）。
