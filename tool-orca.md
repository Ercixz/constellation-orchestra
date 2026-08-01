# Orca 命令速查（详）

> 总览索引见 `SKILL.md`。本文件是 Orca 全命令面提炼。

## 启动 / runtime

- `orca open`：启动并等 runtime 可达（多数命令前置）
- `orca serve [--port p] [--pairing-address host] [--mobile-pairing] [--no-pairing] [--project-root p] [--recipe-json]`：无头 runtime
- `orca status [--json]`：app/runtime/graph 就绪度
- `orca diagnostics memory [--json]`：内存快照
- `orca agent-context [--json]`：agent 可读命令 schema
- 远程 runtime：`orca environment add --name <n> --pairing-code <code>`；或用 `ORCA_PAIRING_CODE`/`ORCA_ENVIRONMENT`

## 账户 / skill

- `orca account add --agent claude|codex`、`orca account list`
- `orca skills list` / `skills get <name>` / `skills install` / `skills update`

## Repo / Worktree（需 git）

- `orca repo add --path <p>` / `repo list` / `repo show` / `repo set-base-ref --ref <r>` / `repo search-refs --query <q>`
- `orca worktree list [--repo R] [--limit n]` / `show --worktree <sel>` / `current [--json]` / `ps [--limit n]`
- `orca worktree create --name <n> [--repo R|--project id [--host id]|--project-host-setup id] [--agent id] [--prompt t] [--base-branch ref] [--issue n] [--linear-issue id] [--parent-worktree sel|--no-parent] [--activate]`
- `orca worktree set --worktree <sel> [--display-name n] [--issue n] [--comment t] [--workspace-status id]`
- `orca worktree rm --worktree <sel> [--force] [--run-hooks]`

> 本场禁 `git init`：不新建 worktree，只用 `orca worktree current` 评估当前目录。

## 文件

- `orca file open <path> [--worktree sel]`
- `orca file diff <path> [--staged]`
- `orca file open-changed [--mode edit|diff|both]`

## 终端

- `orca terminal list [--worktree sel]` / `show --terminal <h>` / `read --terminal <h> [--cursor n] [--limit n]`
- `orca terminal create [--worktree sel] [--title n] [--command c] [--focus]`
- `orca terminal send --terminal <h> --text <t> [--enter] [--interrupt]`
- `orca terminal wait --terminal <h> --for exit|tui-idle [--timeout-ms ms]`
- `orca terminal stop --worktree <sel>` / `split` / `switch` / `focus` / `close [--tab]` / `rename`

## 编排（task→dispatch→gate→worker→message）

- Run：`orchestration run-create` / `run-use` / `run-current` / `run-list` / `run-show`
- 任务：`orchestration task-create` / `task-list` / `task-update`
- 派单：`orchestration dispatch` / `dispatch-show`
- 消息：`orchestration send` / `check` / `ask`（阻塞问协调者）/ `reply` / `inbox`
- 门禁：`orchestration gate-create` / `gate-resolve` / `gate-list`
- worker：`orchestration worker-start` / `worker-show` / `worker-read` / `worker-stop` / `worker-abandon`
- 协调：`orchestration coordinator-start` / `coordinator-stop`（legacy 自动协调循环）
- 重置：`orchestration reset`

## Computer Use / 模拟器

- `orca computer capabilities` / `permissions` / `list-apps` / `list-windows` / `get-app-state` / `click` / `perform-secondary-action` / `scroll` / `drag` / `type-text` / `press-key` / `hotkey` / `paste-text` / `set-value`
- `orca emulator list` / `attach <device>` / `tap <x> <y>`（归一化 0..1）/ `type <text>` / `gesture <json>` / `button <name>` / `rotate <o>` / `exec --command` / `kill`

## 浏览器

- 标签：`tab create [--url]` / `tab list` / `tab show [--page]` / `tab current` / `tab switch [--index|--page]` / `tab close`；profile：`tab profile list|create|delete|set|show|use-default|clone`
- 导航/读取：`goto --url` / `back` / `forward` / `reload` / `snapshot` / `get --what <text|html|value|url|title>` / `is --what <visible|enabled|checked>` / `eval --expression <js>` / `wait [--timeout ms]`
- 交互：`click --element <ref>` / `dblclick` / `fill --element <ref> --value <v>` / `type --input <t>` / `select` / `hover` / `keypress --key` / `focus` / `clear` / `drag --from --to` / `upload --files` / `scroll --direction up|down --amount n` / `scrollintoview`
- 设备/网络：`set device --name "iPhone 12"` / `set offline --state on|off` / `set headers` / `set credentials` / `set media --color-scheme` / `mouse move|down|up|wheel` / `clipboard read|write` / `dialog accept|dismiss` / `storage local|session get/set/clear` / `download --selector --path` / `highlight` / `inserttext` / `find --locator <role|text|label> --value` / `screenshot --format png|jpeg`
- `orca exec --command "..."`：任意 browser 命令透传

## Selector 规则

- `--repo`：`id:<id>` | `name:<name>` | `path:<path>`
- `--worktree`：`id:<repo>::<path>` | `name:<displayName>` | `branch:<branch>` | `issue:<number>` | `path:<path>` | `active`/`current`
- `--terminal <handle>`：`orca terminal list --json` 返回的运行时 handle
- 发现用 selector，重复操作用 handle

## 工作流

1. **隔离任务**（git 环境）：`worktree create --name <t> --agent <id> --prompt "<任务>"`；或 `terminal create --worktree active --command "codex"` 在当前 worktree 起 agent。
2. **本场无 git**：只 `orca worktree current` 评估。
3. **编排闭环**：task-create → dispatch → worker-start → gate-create（决策门）→ gate-resolve → worker-read 验收。
4. **浏览器流程**：tab create/goto → snapshot（拿 `@e1` ref）→ click/fill/keypress → 导航后**必须重新 snapshot**（ref 会变）。
