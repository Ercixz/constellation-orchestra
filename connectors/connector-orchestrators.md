# Connector：如何连接编排器（Orchestrators）

> 编排器 = 组织/调度/派发工作的环节。接入本 skill 的三个编排器：Orca、CMUX、Paseo。
> 各编排器的完整命令面与"四问"详解见 `orchestrators/orca.md`、`orchestrators/cmux.md`、`orchestrators/paseo.md`。
> 本文件是"接入总表"——快速定位怎么连、前提是什么、怎么互相协作。

## 一、三编排器接入总表

| 编排器 | CLI 命令 | 启动前提 | 认证/连接 |
|---|---|---|---|
| **Orca** | `orca` | 需 runtime 已 open（`orca open` 或 app 在跑）；多数命令前置 | runtime socket；远程 runtime 用 pairing code / `ORCA_ENVIRONMENT` |
| **CMUX** | `/Applications/cmux.app/Contents/MacOS/cmux`（PATH 无 cmux，用全路径） | app 常驻即可 | socket `~/.local/state/cmux/cmux.sock`；`CMUX_SOCKET_PASSWORD` / `--password` |
| **Paseo** | `~/.local/bin/paseo`（daemon 侧） | 需 daemon 运行（`paseo daemon status` / `curl localhost:6767/api/health`） | daemon API `127.0.0.1:6767`；`PASEO_HOME`/`PASEO_LISTEN` 可覆盖 |

> 共性：**编排器都是"常驻/需要先拉起"的服务**。连不上先查前提（runtime/daemon/app），再查 socket/端口。

## 二、编排器之间如何协作（互连方式）

编排器不是孤岛，它们通过"共享现场"与"消息"互连：

### CMUX ↔ Orca
- CMUX surface 里开 Orca runtime：在任意 surface 运行 `orca open` / `orca serve`，Orca 即在该面板内可用。
- Orca worktree 的 terminal 本质是 CMUX/终端面板的一部分：`orca terminal create` 拉起的终端，可出现在 CMUX 布局里。
- 协作模式：CMUX 管"多面板并行"（每个面板一个 agent），Orca 管"某个 worktree 内的隔离开发"。两者可同屏并存。

### CMUX ↔ Paseo
- 派单入口在 CMUX，回传入口在 Paseo：Manager 在 CMUX surface 派单 → worker 干完 `paseo send <manager-id>` → Manager 的 Paseo 会话自动收到回传。
- 同一 surface 的 agent 可用 Paseo 守护生命周期（长期任务/定时/心跳）。

### Orca ↔ Paseo
- Orca 的 `worktree create --agent <id>` 拉起 agent，Paseo 可对同一 agent 做守护/定时/跨会话管理。
- 目标架构（见 philosophy.md）：**Paseo 主管 Agent ↔ 用户对话，主管分发任务到 Orca 子 Agent**（每个子任务一个 worktree）。

### 通用协作骨架

```
编排器 A（如 CMUX）  派单 ──►  worker（一个 agent）
        ▲                 │ 干完
        │                 ▼
        └────── paseo send 回传 ◄── Paseo（主管/接收方）
```

## 三、选型速判

| 你要什么 | 用哪个编排器 |
|---|---|
| 快速并行派多个运维/短任务 | CMUX（多面板） |
| 隔离开发、worktree、内置任务系统 | Orca |
| 长期任务、定时、心跳、跨会话守护 | Paseo |
| 主管对话 + 拆任务派给子 Agent | Paseo 主管 + Orca 子 Agent（目标架构） |

## 四、接入检查清单

1. 先读对应 `orchestrators/*.md` 的"四问"，确认 CLI 连接命令与前提。
2. 验证健康：`orca status` / `cmux ping` / `paseo daemon status`。
3. 无凭据明文；socket/密码走本机 600 权限配置。

---

相关文件：`orchestrators/orca.md`、`orchestrators/cmux.md`、`orchestrators/paseo.md`、`philosophy.md`。
