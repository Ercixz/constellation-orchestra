# domain-modeling（共享语言 / 领域建模）

## 定位

> 一句话：主动建/磨项目的**领域模型**——挑战术语、造边界场景、术语定了立刻写进 `CONTEXT.md`（glossary）+ `docs/adr/`（决策）。

## 给谁用

- **Manager**（保证团队共享语言，让研究员产出能对齐）。

## 何时用

- 团队用词不一致、术语模糊/过载（一个词干三个活）。
- 出现了难反悔的架构/方案决策，需要记 ADR。
- 另一个 skill（如 grill-with-docs）需要维护领域模型时自动拉入。

## 关键纪律

1. **CONTEXT.md 只当 glossary**——绝不掺实现细节、不当 spec、不当草稿。
2. **挑战冲突术语**：用户用词与 glossary 冲突 → 立刻指出"你定义 X 是 A，但你像在说 B，哪个？"
3. **磨糊词**：过载词给精确规范名（"你说的 account 是 Customer 还是 User？"）
4. **造边界场景**压测概念边界；**对照代码**查矛盾（"代码整单取消，你说可部分取消——哪个对？"）
5. **ADR 三条件才写**：难反悔 + 无上下文会惊讶 + 真有 trade-off。缺一不写。
6. **就地即写**：术语定了立刻更新 CONTEXT.md，不批量攒。

## 原文链接

https://github.com/mattpocock/skills/tree/main/skills/engineering/domain-modeling
