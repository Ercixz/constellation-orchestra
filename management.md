# 管理（Management）— 系统如何组织与运转

> 本文档是 **管理器/管理方式** 章节：讲"系统怎么组织、任务怎么流转、组件怎么分工"。
> 它与 `philosophy.md`（哲学：二元论/可插拔/世界观）区分——哲学回答"为什么"，管理回答"怎么跑"。
> 完整设计稿：`AI-Persistence/Memory-Bucket/2026-08-01-System-Architecture-GitHub-Task-Platform.md`（本文是方法论摘要，细节在知识库）。

## 一、系统架构：GitHub 任务平台

本场（Endpoint-Cluster-Console 及其未来所有项目）的目标架构：**GitHub Issues 作任务载体 + Orca 编排 + 多 Agent 协作 + wayfinder 规划**。

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

## 二、与哲学的衔接

- 四层 = 二元论的具体化（任务层+介质层是"内容"，编排层+执行层是"环节"）——哲学见 `philosophy.md`。
- 轻/中型直连协议（`philosophy.md` 第六节"团队协作分派"）是这套架构的轻量档；重型（wayfinder 规划 + 流程框架）是重量档前置。
- 编排重量分级（轻/中/重）是哲学层面的"任务该多重"，本文件是"定了重量之后系统怎么跑"。
- 介质层保留 tickets/receipts 文件制——轻量任务不强制上 GitHub。

---

相关文件：`SKILL.md`（总纲）、`philosophy.md`（哲学：二元论/可插拔/重量分级/轻重分离）、`protocol-cmux.md`（派单协议）。
