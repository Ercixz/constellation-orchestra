---
name: wayfinder
description: 大型任务规划：把超过一次 agent session 的大块工作画成决策票地图，一次一票解决直到路清晰。Manager 专用，中心化派单。
disable-model-invocation: true
---

# wayfinder — 大型任务规划（决策票地图）

一个**模糊的、大到一次 agent session 装不下**的想法来了，路线裹在迷雾（fog）里：从现在到**目的地（destination）**的路还看不清。wayfinder 的职责是**把路画出来**，而不是直奔目的地。它把这条路线画成一张 repo 上的**共享地图（map）**，然后一张一张地解**决策票（decision tickets）**——票的产出是**决策**，不是要执行的构建切片——直到路清晰为止。

目的地随每次任务而变，**命名目的地是画图的第一步**——它决定每一张票的形状。目的地可能是一份待交付/迭代的 spec、一个规划前必须锁死的决策、或一处就地改造（如数据结构迁移）。地图是领域无关的——工程、课程内容，什么形态都适用。

> 本版为 **Project Deterritorialization** 本地化改造版：原版为去中心化认领制，本版改为 **Manager 中心化派单制**，并定制 HITL 上报、Local Markdown Tracker、层级通信三条项目纪律。核心概念与英文术语保留原版。

## Plan, don't do（规划归规划，执行归执行）

wayfinder 默认只做**规划**：每张票解决一个决策，当地图走到"路已清晰——不再有任何决策待解"即完成。想直接去干活的冲动，通常是"你已走到地图边缘、该交接执行"的信号。任务可通过其 **Notes** 覆盖这一默认（把执行也带进地图），但默认情况下：**产决策，不产交付物**。

## Refer by name（用名字引用）

每张地图和每张票都有**名字（title）**。在一切人类可读处——叙述、地图的 Decisions so far——**用名字引用，绝不裸用 id/编号/slug**。一面墙的 `#42, #43, #44` 无法阅读；名字一眼可读。id 和链接不消失——名字包裹链接——但只**藏在名字里面**，不顶替名字。

## The Map（地图）

地图是 repo issue tracker 上的一张 issue，带 `wayfinder:map` 标签，是**唯一权威产物（canonical artifact）**。它的票是地图的子 issue。

地图是**索引（index），不是仓库（store）**：它列出已做的决策、指向承载细节的票；一个决策只存在于一处——它的票——所以地图绝不重述它，只摘录并链接。

**地图、子票、blocking、frontier 查询的物理载体依 tracker 而定。** 应已提供 tracker；未提供时，默认落到 **Local Markdown Tracker**（见下，本项目即用此落地）。

### 地图正文

整张地图低分辨率一次载入（每 session 一次）。**打开的票不列在地图里**——它们是开放的子 issue，靠查询发现。

```markdown
## Destination

<走完这张地图长什么样——这个 effort 要找到的 spec/决策/改动。一两行；每个 session 先用它定向，再选票。>

## Notes

<领域；每个 session 该 consult 的技能；本 effort 的既有偏好>

## Decisions so far

<!-- 索引——每个已关票一行：足够判断相关性，再放大链接看细节 -->

- [<已关票名>](link) — <答案的一句话摘录>

## Not yet specified

<!-- 见「Fog of war」：范围内但还没清晰到能开票的雾；frontier 前进时毕业成票 -->

## Out of scope

<!-- 见「Out of scope」：被裁定超出目的地的活；关闭，永不毕业 -->
```

### 票（Tickets）

每张票是地图的**子 issue**；tracker 的 issue id 是它的身份。正文是**问题**，大小适合一次 100K token 的 agent session：

```markdown
## Question

<这张票要解决的决策或调查>
```

每张票带 `wayfinder:<type>` 标签——`research` / `prototype` / `grilling` / `task` 之一（见 [票型](#ticket-types)）。

**派单（本版替换认领）**：一张票由 **Manager 明确派给**某位 Worker/Researcher（写入派单记录），任何工作开始**之前**先派——一张开放未派单的票就是未开工。**Manager 一次只推一张 frontier 票**，不并行认领（与原版"并发 session 各自认领"不同）。

Blocking 用 tracker 的**原生依赖关系**——关键在于它把 frontier **视觉化**呈现在 tracker 自己的 UI 上，人类不打开地图就能看到哪些可拿。只有缺少原生 blocking 的 tracker 才退回正文约定。一张票**解阻（unblocked）**当所有 blocking 它的票都已关闭；**frontier** 就是所有开放、解阻、未派单的子票——已知的边缘。

答案不在正文里——在**解票时记录**（见「Work through the map」）。解票过程中产出的资产从 issue 链接，不粘贴进来。

## Ticket Types（票型）

每张票要么是 **HITL**——人参与其中，一个能为自己发言的人陪着做——要么是 **AFK**，agent 独自行驶。HITL 票只有通过那次真实交换才能解决；agent 绝不替人类说话（一个自己回答自己问题的 grilling agent 就是坏了）。

- **Research（AFK）**：读文档、第三方 API、或本地知识库等资源，浮出某个决策等的事实。由 `/research` **subagent** 解决。当需要当前工作目录之外的知识时用。
- **Prototype（HITL）**：做一个廉价、粗糙、具体的产物给人反应，把讨论的保真度提上来——大纲、粗略想法、stub、或 UI/逻辑代码（经 /prototype 技能）。把原型作为资产链接。当关键问题是"应该长什么样 / 应该怎么表现"时用。
- **Grilling（HITL）**：经 /grilling 与 /domain-modeling 技能，一次一个问题地对话。**默认票型**。
- **Task（HITL 或 AFK）**：某个决策能做**之前**必须发生的**手动活**——无可决策、无可原型、无可调研，但讨论被它卡住。签约一个服务好评估其 API、申请权限、搬数据好看到它的形状。这是唯一一个**做**而不是**决定**的票型——它凭"解阻一个决策"挣得席位，而不是凭交付目的地。agent 能独立干就独立干（AFK）；否则给人一张精确清单（HITL）。活干完即解；答案记录做了什么、以及后续票依赖的事实（凭据位置、新 URL、行数）。

## Fog of war（雾）

地图**刻意不完整**：别画你还看不见的东西。活跃票之外是**雾**——你能看出将出现、但还钉不下来的决策和调查，因为它们挂在仍开放的问题上。解一张票会清掉它前面的雾，把现在可specify 的东西毕业成新票——一次一张，直到通往目的地的路清晰、无票可开。

地图的 **Not yet specified** 区就是写下那片模糊视野的地方：疑似问题、以后要回头看的区域。它是**朝目的地**的未发现前沿——这里的一切都在范围内，只是还不够尖到能开票。按视野允许的松散或完整程度写；它同时是合作者读"这个 effort 往哪去"的路标。

**雾还是票？** 判据是你**现在**能否精确陈述这个问题——**不是**你现在能否回答它。

- **开票当**：问题已经够尖——即使它被阻塞、你还动不了它。
- **Not yet specified 当**：你还不能把它说得那么尖。别把雾预先切成票大小的片：它比票粗，一片雾可能在 frontier 到达后毕业成好几张票，或一张也不。

**Not yet specified 排除**：已决定的（Decisions so far）、已是活跃票的、以及超出范围的（下一节）。

## Out of scope（超出范围）

雾只朝**目的地**聚集。目的地锁死范围，所以超出它的活是 **out of scope**——它**不是**雾，不属于 **Not yet specified**。它在地图上有自己的 **Out of scope** 区：你有意识地把**这个 effort** 排除的活。是**范围**（不是锐度）把它放进来。

Out-of-scope 的活**永不毕业**——frontier 在目的地停下——所以只有目的地被重画它才会回来，而且是作为新 effort，不是恢复。

裁定某活超出范围是**界定范围**的动作，不是路线上的一步。当一张已存在的票落到了目的地之外——画图时错划进来，或被一次解票暴露——就**关闭它**（关闭票明确离开 frontier），并在 **Out of scope** 区留一行：摘录 + 为什么超范围，链接那张已关票。它不进 **Decisions so far**——那记录真正走过的路线；一条范围边界不是路线上的步。

## 本项目定制（三条铁律）

### HITL 上报机制（人不直接对接 Worker/Manager）

grilling / prototype 票需要人参与时，**Manager 不直接找用户**。流程：

1. Manager 写**上报单**（问题 + 选项 + 背景），传给秘书。
2. 秘书与用户对话，把答案回传 Manager。
3. Manager 收到答案后**解票**继续。

> 对齐协议 §5.5 + §2.6 层级通信铁律：**人只和秘书交谈**；Manager/Worker 不对用户；Worker/Researcher 不直接找秘书。

### Local Markdown Tracker（未接 GitHub 时的落地）

无 GitHub 时用本地文件制，地图与票都是 markdown 文件，路径对齐协议 §5.5：

```
agent-journal/wayfinder/<feature>/
├── map.md              ← 地图（Destination / Notes / Decisions so far / Not yet specified / Out of scope）
└── tickets/
    ├── ticket-001-grilling-<topic>.md   ← 每票一文件：问题 / 类型 / blocking / 状态
    ├── ticket-002-research-<topic>.md
    └── ...
```

- **地图是唯一真相（source of truth）**：状态全在文件里，不依赖任何会话记忆。
- **Manager 会话可短命**：每次推进 = 拉起 → 读 map.md → 解 frontier 票 → 产物落盘 → 关会话。
- frontier = 所有 blocker 已完成、未派单的票；关闭票 = 在 map.md 的 Decisions so far 加一行（**用名字引用**，不用裸 id）。

### 层级通信（Manager 中心化派单与回执）

- **Manager ←→ Worker/Researcher**：双向派单/回执；Manager 画地图拆票，明确派给 Worker/Researcher，**无认领制**。
- **秘书 ✗→ Worker/Researcher**：秘书不直接对 Worker 发指令/催办。
- **Worker/Researcher ✗→ 秘书**：回执只写 submission 文件给自己的 Manager，不直接汇报秘书。

## Invocation（调用）

两种模式。无论哪种：**每个 session 最多解一张票**——research 票除外。

### Chart the map（画地图）

用户拿一个模糊想法来调用。

1. **命名目的地。** 跑一次 `/grilling` 和 `/domain-modeling` session，钉下这张地图在找什么——spec、决策、改动。目的地锁死范围，所以先定它。
2. **画 frontier。** 再 grill 一次，这次**广度优先**：在整个空间里扇开而不是深挖任何一条线，浮出开放决策和现在能走的第一步。**如果浮不出任何雾**——去目的地的路已经清晰、整段旅程小到一次 session 装得下——你不需要地图。停下，问用户想怎么继续。
3. **创建地图**（`wayfinder:map` 标签）：填 Destination 和 Notes，Decisions so far 留空，把雾草进 **Not yet specified**。
4. **创建你现在能 specify 的票**，作为地图的子 issue——然后**第二遍**接 blocking 边（issue 需要 id 才能互相引用）。接线把它们排进 frontier 和被阻塞；一切现在还 specify 不了的留在雾里——**Not yet specified** 区。
5. **派发 research 票。** 对刚创建的每张 `research` 票，由 Manager 派 `/research` subagent 并行解决，在一次性 `research/<name>` 分支上捕获发现，票上带上下文指针。
6. 停下——画地图是一次 session 的活；它不手工解任何票。

### Work through the map（解地图）

用户拿地图（URL 或编号）来调用。票是**可选**的——没有时，由 Manager 选下一张 frontier 票，不是用户选。

1. 载入**地图**——低分辨率视图，不是每张票的正文。
2. 选票。用户点名了就用它；否则取 frontier 里第一张。**（本版）Manager 将其派给对应的 Worker/Researcher**，任何工作开始前先派单。
3. 解它——**按需放大**：按需抓任何相关或已关票的完整正文；调用 Notes 块点名的技能。拿不准就用 `/grilling` 和 `/domain-modeling`。
4. 记录解票：把答案发成**解票评论**，**关闭** issue，并把**上下文指针**append 到地图的 Decisions so far。
5. 补新增票（create-then-wire）；把答案已使之可 specify 的雾毕业成新票，把每个毕业的 patch 从 **Not yet specified** 清掉，让它只活在它的新票里。如果答案揭示一张票——这张或其他——落在目的地之外，就把它**裁定为 out of scope**，而不是在路线上解它。如果决策使地图其他部分失效，更新或删除那些票。

用户可并行跑解阻票，所以可能看到其他 session 同时在改 tracker。

## 与相关技能的关系

- **上游原语**：grilling / grill-with-docs（grilling 票的追问原语）、research（research 票）、prototype（prototype 票）。
- **下游**：to-tickets（票清后拆实施工单，进入正常 dispatch → submission → acceptance 闭环）。
- **不是任何技能的"升级版"**：wayfinder 是规划层容器，grilling / grill-with-docs / research / prototype 是执行层原语——层级不同，互补。
- 本体系落地要点见 `skill-routes/wayfinder.md`（浓缩版，保留不动）。
