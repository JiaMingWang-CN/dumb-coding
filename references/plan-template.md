# 增量实施计划（Plan）标准模板与交接协议

本文档供 `dumb-coding` 技能的 `ask-plan` 子阶段（[ask-plan.md](./ask-plan.md)）在编写实施计划文档及发起出口门禁审阅交接时按需读取。

---

## 1. 实施计划（Plan）标准文档模板

每份实施计划必须以如下标准头部开篇，随后按顺序排列自包含的增量任务（`Task 1` .. `Task N`）：

> **门禁状态契约**：初稿落盘时状态填为 `待批准（YYYY-MM-DD）`，`Execution Mode` 填为 `待批准时由用户选定`；仅在用户批准 Plan 并选定执行模式后，**一次性**更新为 `已批准（YYYY-MM-DD）` 并写入对应的执行与终审协议，此后整个执行期 Plan 原文保持只读。

````markdown
# [特性 / 模块名称] 实施计划

- **状态**：待批准（YYYY-MM-DD） | 已批准（YYYY-MM-DD）

> **致执行代理 / 工程师（For Agentic Workers）**：**本 Plan 为只读蓝图，请使用宿主原生任务列表追踪推进，严禁回写修改本文档。** 请严格按任务编号顺序与下方 `Execution Mode` 协议实施。每个任务均隐式包含 `Global Constraints`；相邻任务仅通过各自 `Interfaces` 块声明的精确名称与类型契约衔接；若某步实际输出与 `Expected` 不符，必须停下排障修复后再继续。

**Goal:** [一句话精准定义本计划交付什么可验证的工程能力]

**Architecture:** [2–3 句话概述核心模块划分、数据/控制流向与关键技术路径]

**Tech Stack:** [涉及的语言、运行时版本、核心库与测试/构建工具]

**Spec:** `[指向已获用户批准的 docs/specs/YYYY-MM-DD-<topic>-design.md 路径]`（本计划的一切决策均锚定该规格，执行与终审阶段须同时参照该 Spec 与本计划）

## Execution Mode

[初稿填“待批准时由用户选定”；获批后按用户选择二选一一次性填入下方对应标准协议：]

- **Subagent-driven（子代理逐任务隔离驱动）**：每推进一个 Task，派全新 Implementer 子代理仅按头部 `Global Constraints`、`Spec` 相关上下文与当前 Task 实施并跑通验证；完成后派全新 Reviewer 子代理对照 `Interfaces`、`Global Constraints` 与测试独立审查，修复阻塞项再进下一 Task；全部完成后派全新 Reviewer 子代理对照 `Spec` 与 `Review Focus` 做全量独立终审。
- **Native + 独立终审（当前会话原生执行）**：按任务与步骤顺序配合宿主原生任务列表逐条实施，遇 `Expected` 不符即停下修复；全部 Task 完成后派高能力模型的全新 Reviewer 子代理对照 `Spec` 与 `Review Focus` 做全量独立终审。

## Global Constraints

[从 Spec 与仓库规范中逐字提取的项目级全局硬约束——每行一条，保留精确版本号、依赖禁令、命名约定、隐私/隔离红线与兼容性要求。所有 Task 均无条件受本节约束。]

- [约束 1：例如 Node.js >= 24.9.0，全量通过 TypeScript strict / exactOptionalPropertyTypes 检查]
- [约束 2：例如 零新增生产运行时依赖；强敏感凭据显式写入直接拒绝]

## Review Focus

[对照 Spec 提炼出“规格未逐一明写、但在真实运行中最可能坑到用户或破坏系统不变量”的前 5 类边界输入、并发竞态或故障模式——按发生概率与危害从高到低排列，每行写清触发条件、合理预期行为，以及负责用测试钉死它的归属 Task 编号。其中，纯工程健壮性防御直接落入对应 Task 测试断言；涉及产品行为语义空白的边界不得擅自写死，须先澄清回补 Spec 或在审批时高亮交由用户确认。]

1. **[边界/故障场景 1]**：[触发输入或竞态条件] → [预期安全行为]。（由 Task X 测试钉死）
2. **[边界/故障场景 2]**：[触发输入或异常条件] → [预期安全行为]。（由 Task Y 测试钉死）

---

### Task 1: [最小可独立测试与审查的组件/能力名称]

**Files:**
- Create: `src/exact/path/to/new-module.ts`
- Modify: `src/exact/path/to/existing.ts`（聚焦修改的真实符号或行号区间，如 `L45-L82`）
- Test: `tests/unit/exact-module.test.ts`

**Interfaces:**
- Consumes:
  - `src/existing/types.ts`: `ExistingConfig`, `normalizeKey(raw: string): string`
- Produces:
  - `src/exact/path/to/new-module.ts`:
    - `export interface ParseInput { readonly raw: string; readonly maxBytes?: number }`
    - `export function parseRecord(input: ParseInput): ParsedRecord`（超限时抛出 `ERR_RECORD_TOO_LARGE`）

- **Step 1: 编写覆盖核心契约与 Review Focus 边界的失败测试**

Create `tests/unit/exact-module.test.ts`:
```ts
import { describe, expect, it } from 'vitest';
import { parseRecord } from '../../src/exact/path/to/new-module.ts';

describe('parseRecord', () => {
  it('parses valid payload and enforces exact maxBytes ceiling', () => {
    expect(parseRecord({ raw: 'key=42', maxBytes: 16 })).toEqual({ key: '42' });
    expect(() => parseRecord({ raw: 'x'.repeat(17), maxBytes: 16 })).toThrow('ERR_RECORD_TOO_LARGE');
  });
});
```

- **Step 2: 运行测试并确认按预期原因失败**

Run: `pnpm exec vitest run tests/unit/exact-module.test.ts`
Expected: FAIL（提示无法解析 `new-module.ts` 或 `parseRecord` 未定义）

- **Step 3: 在 `src/exact/path/to/new-module.ts` 中按 `Interfaces` 精确签名完成最小实现**

[当签名与测试断言已唯一确定实现时，此处无需贴函数体；仅当存在库调用选择、特殊状态转移或复杂算法分歧时，用 1–3 行说明选型理由或附上核心算法片段。]

- **Step 4: 运行测试与类型检查确认通过**

Run: `pnpm exec vitest run tests/unit/exact-module.test.ts && pnpm run typecheck`
Expected: 全部 PASS，零类型错误

- **Step 5: 提交本任务增量**（仅 Git 仓库保留；非 Git 仓库删除本步）

```bash
git add src/exact/path/to/new-module.ts tests/unit/exact-module.test.ts
git commit -m "feat(module): add bounded record parser"
```
````

---

## 2. 出口门禁审阅与执行方式交接话术

完成六维自审并保存计划后，在正文中给出计划文件链接与你推荐的执行方式，并通过结构化提问工具（`ask_question` / `AskUserQuestion`）请求用户审阅批准：

### 场景 A：用户尚未预先指定后续执行方式
在消息中清晰说明：
1. 计划已保存至 `docs/plans/<filename>.md`（附可点击链接）并通过六维自审；
2. 简述后续下一阶段可选的两种分层执行模式及其权衡：
   - **子代理逐任务隔离驱动（Subagent-driven）**：每个 Task 派发全新上下文的 Implementer 子代理仅按头部全局约束与自身 Task 编码，完成后派发全新 Reviewer 子代理独立审查契约与测试，通过后再推进下一 Task，收尾再做全量 Spec + `Review Focus` 终审。上下文零污染、质量最高，消耗更多子代理轮次。
   - **当前会话按步原生执行（Native + 独立终审）**：在执行会话中配合宿主原生任务列表按步推进、遇预期不符即停下修复，全部任务完成后派发一个强模型全新 Reviewer 子代理对照 Spec 与 `Review Focus` 做全量独立终审。速度最快、开销最省。
3. 结合本计划的**任务数量、跨任务接口耦合度与出错返工代价**，明确给出你的首选推荐及一句话理由；
4. 调用结构化提问工具请用户确认：**（a）批准该实施计划并选定执行模式（获批后将一次性更新 Plan 头部状态为 `已批准` 并写入所选 `Execution Mode` 协议，随后结束 `ask-plan` 阶段，方便在当前会话或新会话直接按该文件执行）；或（b）指出需要修订的计划细节。**

### 场景 B：用户此前已明确指定了执行方式
在消息中附上 `docs/plans/<filename>.md` 链接并说明后续将保留其指定的执行方式，调用结构化提问工具请用户审阅确认计划是否准确锁定需求（批准后一次性更新头部状态为 `已批准` 并写入所选 `Execution Mode` 协议，随后立即结束 `ask-plan` 阶段，等待下一条执行指令）。
