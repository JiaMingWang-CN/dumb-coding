# dumb-coding — 傻子编程法

> **承认人和 AI 都会犯傻：在写第一行代码之前，先把话问明白、把图纸画死。**

`dumb-coding` 是一个受 [superpowers](https://github.com/obra/superpowers) 启发、结合真实 AI 编程踩坑经验重构而成的 **Agent Skill**。针对 AI 极其擅长“听半句话就开始自信瞎写三百行”以及人类提需求时常常“以为自己说清楚了、其实边界全没想”的双向痛点，把流程拆成两个带强硬门禁的阶段：**先 `ask` 对齐 Spec，再 `ask-plan` 锁死工程蓝图**。

---

## 为什么叫“傻子编程”？

常规 Vibe Coding 默认模型和人都足够聪明；`dumb-coding` 假定只要不设防呆护栏，两边早晚都会翻车：

- **防 AI 盲问与臆造 API**：开口提问和拆解任务前强制先查代码库、调用链与依赖锁文件（Ground Before Asking），仓库里能查到的事实绝不反问用户。
- **防隐含边界与“暗度陈仓”默认**：用单步结构化提问挖出真实动机（Why）、非目标与反目标（Anti-Goals），所有 AI 自行拍板的工程默认（Builder Calls）必须显式亮明。
- **防 Plan 沦为“比源码还长的代码抄本”**：`ask-plan` 只锁定实现者无法独自决定的精确工程决策（文件边界、跨任务 `Interfaces` 契约、TDD 断言、前 5 类隐含故障模式 `Review Focus`），把具体函数体留给执行者。
- **防批准后顺手越界狂写代码**：两阶段各有 `<HARD-GATE>`——Spec 获批即停、Plan 获批并写入执行协议即停，绝不越界抢跑。

---

## 两阶段工作流

```text
需求输入
  │
  ├─► 零歧义微调（改文案/色值等）──► 直接完成
  │
  └─► 非琐碎工程变更
        │
        ▼
  [阶段 1: ask] 事实接地 ─► 单步追问 ─► 方案比选 ─► (完整工程) Builder Check 盲审
        │
        ▼ 审批锁定 docs/specs/YYYY-MM-DD-<topic>-design.md (状态: 已批准)
        │
  [阶段 2: ask-plan] 二次接地 ─► 文件映射 ─► Interfaces + TDD ─► 六维自审
        │
        ▼ 审批锁定 docs/plans/YYYY-MM-DD-<feature-name>.md (状态: 已批准 + Execution Mode)
        │
  [后续执行] 当前会话或新开干净会话按 Plan 头部 Execution Mode 只读执行 + 原生任务列表追踪
```

---

## 仓库结构

本仓库遵循 [Agent Skills](https://agentskills.io) 开放标准，采用严格单技能 + 渐进披露结构：

```text
dumb-coding/
├── SKILL.md                             # 技能统一入口（阶段路由、产物契约与总硬门禁）
├── references/
│   ├── ask.md                           # 阶段 1 手册：事实接地与需求规格对齐
│   ├── spec-and-builder-check.md        # 阶段 1 模板：Spec 标准模板与 Builder Check 盲审提示词
│   ├── ask-plan.md                      # 阶段 2 手册：规格驱动的契约化增量规划
│   └── plan-template.md                 # 阶段 2 模板：Plan 标准模板与交接协议
├── README.md
└── .gitignore
```

---

## 安装与使用

### 安装

1. **让 AI 自动配置（推荐）**
   直接把下面这段英文提示词发给你的 AI 编程助手即可完成安装与校验：
   ```text
   Install and configure the `dumb-coding` Agent Skill from https://github.com/JiaMingWang-CN/dumb-coding for my coding agent environment.

   1. Check which skills directory this agent/IDE uses (e.g., `.agents/skills/dumb-coding/`, `~/.agents/skills/dumb-coding/`, or `~/.claude/skills/dumb-coding/`). If scope isn't clear, ask me whether to install it for the current project or globally.
   2. Install it via `npx skills add JiaMingWang-CN/dumb-coding`, or if that isn't available, clone/copy the repository's `SKILL.md` and `references/` directory into the target `dumb-coding/` skills folder.
   3. Verify that `SKILL.md` (`name: dumb-coding`) and all four files under `references/` (`ask.md`, `spec-and-builder-check.md`, `ask-plan.md`, `plan-template.md`) are in place and discoverable, then confirm the installed path and how to invoke it.
   ```
2. **通过 Skills CLI 安装**
   ```bash
   npx skills add JiaMingWang-CN/dumb-coding
   ```
3. **手动放入宿主技能目录**
   将整个 `dumb-coding/` 目录复制或软链接到你所用 Agent 的技能目录中，例如：
   - 项目级：`<your-project>/.agents/skills/dumb-coding/`
   - 全局级：`~/.agents/skills/dumb-coding/` 或 `~/.claude/skills/dumb-coding/`

### 使用方式

- **阶段 1（对齐需求，锁定 Spec）**：直接描述你想做什么或要改什么，或显式提及使用 `dumb-coding`；Agent 会先做代码调研，再通过单步提问带你收敛并批准 `docs/specs/...-design.md`。
- **阶段 2（制定计划，锁定蓝图）**：Spec 获批后，在下一条指令中说明“按 `docs/specs/...-design.md` 制定实施计划”；Agent 会产出零歧义 Plan 并在你批准时一次性锁定 `Execution Mode`（`Subagent-driven` 子代理逐任务隔离驱动 / `Native + 独立终审`）。
- **开始编码**：在当前会话或新开干净会话中发送“按 `docs/plans/...md` 执行”，执行者会严格遵守文件头部的 `Execution Mode` 与只读蓝图约束推进。

---

## 致谢与贡献

- 流程思想灵感源自 [superpowers](https://github.com/obra/superpowers)（`brainstorming` 与 `writing-plans`）。
- 欢迎提交 Issue 或 PR，修改文档时请保持跨文档门禁、模板与术语口径一致。
