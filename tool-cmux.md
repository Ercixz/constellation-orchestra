# CMUX 命令速查（详）

> 总览索引见 `SKILL.md`。本文件是 CMUX 全命令面提炼。

## 启动 / 环境

- CLI 全路径：`/Applications/cmux.app/Contents/MacOS/cmux`
- Socket：默认 `~/.local/state/cmux/cmux.sock`，`CMUX_SOCKET_PATH` 可覆盖
- 鉴权优先级：`--password` > `CMUX_SOCKET_PASSWORD` > Settings
- 终端内环境变量：`CMUX_WORKSPACE_ID`、`CMUX_SURFACE_ID`、`CMUX_TAB_ID`（作各命令默认 target）
- 引用：UUID / 短 ref `window:1/workspace:2/pane:3/surface:4` / 索引；`--id-format uuids|both`
- 配置：`cmux docs settings` / `cmux settings path`；`cmux reload-config` 热重载 Ghostty + `~/.config/cmux/cmux.json`
- 编辑 cmux.json 前先做时间戳 .bak 备份；Ghostty 配置在 `~/.config/ghostty/config`

## 面板 / workspace 生命周期

| 命令 | 说明 |
| --- | --- |
| `cmux <path>` | 目录 → 新 workspace |
| `new-workspace [--name N] [--cwd P] [--command C] [--layout json] [--focus]` | 新建 |
| `list-workspaces` / `current-workspace` / `select-workspace` / `rename-workspace` / `close-workspace` | 查/切/改/关 |
| `new-window` / `list-windows` / `focus-window` / `close-window` / `rename-window` | 窗口 |
| `new-pane [--type terminal\|browser] [--direction d] [--url u] [--focus]` | 新面板 |
| `new-split <left\|right\|up\|down>` | 拆分当前 |
| `new-surface [--type terminal\|browser\|agent-session] [--provider codex\|claude\|opencode] [--renderer react\|solid]` | 新建 surface |
| `split-off --surface S <dir>` / `move-surface --surface S [--pane P]` / `reorder-surface` / `drag-surface-to-split` | 重组 |
| `focus-pane/panel/surface/workspace` | 聚焦 |
| `close-surface` / `close-pane` / `close-workspace` | 关闭 |
| `move-workspace-to-window` / `reorder-workspace[s]` / `move-tab-to-new-workspace` | 布局调整 |
| `workspace-action --action <name> [--color] [--description]` / `workspace status set <lane\|auto>` | workspace 状态/动作 |
| `todo <add\|list\|check\|uncheck\|start\|rm\|clear>` | 轻量 todo |

## 与面板交互（派单核心）

| 命令 | 说明 |
| --- | --- |
| `read-screen [--surface S] [--scrollback] [--lines n]` | 读屏 |
| `send [--surface S] <text>` | 发文本 |
| `send-key [--surface S] <key>` | 按键 |
| `send-panel --panel P <text>` / `send-key-panel` | 对 panel 发 |
| `notify --title T [--body B]` | 通知（可带 workspace/surface target） |
| `markdown open <path>` | markdown 面板（live reload） |
| `diff [patch-file\|-] [--source <unstaged\|staged\|branch\|last-turn>] [--cwd P] [--base ref] [--layout split\|unified]` | diff 面板 |
| `identify [--workspace W]` | 识别调用者 workspace |

## tmux 兼容

`capture-pane`、`resize-pane`、`pipe-pane`、`wait-for [-S name]`、`swap-pane`、`break-pane`、`join-pane`、`next/previous/last-window`、`last-pane`、`find-window`、`clear-history`、`set-hook`、`bind-key/unbind-key`、`set-buffer/list-buffers/paste-buffer`、`respawn-pane`、`display-message [-p]`

## 浏览器自动化

`browser open/open-split/goto/navigate/back/forward/reload`、`browser snapshot [--interactive] [--cursor] [--compact]`、`browser click/dblclick/hover/focus/check/uncheck/type/fill/press/select/scroll`、`browser eval <script>`、`browser wait [--selector/--text/--url-contains/--function/--timeout-ms]`、`browser screenshot [--out]`、`browser get <url|title|text|html|value|...>`、`browser find <role|text|label|...>`、`browser tab <new|list|switch|close>`、`browser cookies/storage/profiles/state/save-load`、`browser console/errors <list|clear>`、`browser devtools/focus-mode/design-mode/zoom`

## 系统 / 状态

`welcome`、`docs <settings|shortcuts|api|browser|agents|dock|sidebars>`、`settings`、`config <doctor|check|validate|reload>`、`shortcuts`、`themes list/set/clear`、`restore-session`、`agent-hibernation on|off`、`disable/enable-browser`、`ping`、`version`、`capabilities`、`events`、`feed tui|clear`、`vm <base|new|ls|status|snapshot|fork|restore|rm|exec|shell|ssh>`、`remotes <list|add|remove>`、`ai-accounts <list|upload|remove>`、`rpc <method> [json]`、`reload-config`、`surface-health`、`debug-terminals`、`trigger-flash`、`refresh-surfaces`

## 通知与状态栏

`list-notifications` / `dismiss-notification` / `mark-notification-read` / `open-notification` / `jump-to-unread` / `clear-notifications`；`set-status/clear-status/list-status`、`set-progress/clear-progress`、`log/clear-log/list-log`、`set-app-focus`、`right-sidebar <toggle|show|...>`、`sidebar <validate|reload|select|open>`、`sidebar-state`

## 本场注意

- 编排 workspace 布局是 3列×2行（opencode TUI 需 ≥60 列），surface 用 UUID 不用数字 ref。
- 派单协议、回执路径、安全纪律见 `SKILL.md` 与 `cmux-orchestration` skill。
