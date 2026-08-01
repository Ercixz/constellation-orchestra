# to-tickets（拆工单）

## 定位

> 一句话：把 plan/spec/对话拆成一组 **tracer-bullet 垂直切片工单**，每个声明 **blocking 边**。

## 给谁用

- **Manager**（拆解工单、派发给研究员/执行者）。

## 何时用

- 需求/规划已清晰，需要切成可逐个执行的工单。
- 需要工单有依赖关系（blocking）可判断"哪个能先开工"。

## 关键纪律

1. **垂直切片**：每片切穿所有层（schema/API/UI/tests），是窄但完整的路径——不是单层的横向切片。
2. **可自证**：每片完成后可独立 demo 或验证。
3. **尺寸 = 单个 context window**：每片够一次全新会话干完。
4. **每个工单给 blocking 边**：哪些工单必须先完成；无 blocker 的可立即开工。
5. **先 prefactor**："先让改动容易，再做容易的改动"——大改动用 expand-contract（先加新形式→分批迁移→删旧形式）。
6. **发布到 tracker**：本地=每票一文件于 `.scratch/<feature>/issues/`；真实 tracker=每票一个 issue + 原生 blocking 链接 + `ready-for-agent` 标签。
7. **frontier** = 所有 blocker 都完成的工单，按此推进。
8. **不关不改父 issue**。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/to-tickets
