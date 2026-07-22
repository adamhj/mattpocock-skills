# Migration Guide — Claude Code Skills → Cline 系平台

本文档定义了将 Matt Pocock Skills（以 Claude Code 为原生平台）迁移到 Cline 系平台（Cline / Roo Code / Zoo Code / Kilo Code）时需要遵循的规则框架和方法论。

## 迁移规则（R1–R13）

以下规则适用于所有 Cline 系目标平台。

### 格式规则

| 规则 | 说明 |
|---|---|
| **R1 — Slash Command 格式** | 目标文件置于 `<platform>/commands/`，单个 `.md` 文件，`---\ndescription: "..."\n---` 格式，无 `name` 字段、无 `disable-model-invocation`。 |
| **R2 — Skill 格式** | 目标文件置于 `<platform>/skills/<name>/SKILL.md`，可附带同级子文件。保留 frontmatter 中的 `name` 和 `description`，无 `disable-model-invocation`。 |
| **R3 — Slash Command 不支持子文件引用** | Slash Command 是单个 `.md`，所有原 skill 引用的子文件（如 `LOGIC.md`、`AGENT-BRIEF.md`）内容必须内联到该 `.md` 中。 |
| **R4 — Slash Command 不支持脚本** | 原 skill 的 `scripts/` 目录不能在 slash command 中使用。但 skill 中保留引用不受影响（Agent 可自行判断）。 |
| **R5 — Skill 不支持 disable-model-invocation** | Cline 系 skill 全部为 model-invoked，无法阻止 Agent 自动触发。原 user-invoked skill 转为 slash command 或接受此行为。 |

### 引用规则

| 规则 | 说明 |
|---|---|
| **R6 — Slash Command 引用其他 Slash Command** | 不可"自动调用"另一个 slash command。原文中的"run `/xxx`"改为"告诉用户运行 `/xxx`"或"建议用户运行 `/xxx`"。 |
| **R7 — Slash Command 引用 Skill** | 保持不变，Skill 名称在 Agent 上下文中可见，Agent 可以自动调用。原文的 `/foo` 改为 `foo` skill。 |
| **R8 — Skill 引用 Slash Command** | 不能"hand off to /xxx"，因为 Agent 看不到 slash command。改为"建议用户运行 `/xxx`"。 |
| **R9 — Skill 引用 Skill** | 保持不变。 |

### 迁移范围规则

| 规则 | 说明 |
|---|---|
| **R10 — 排除 personal** | `skills/personal/` 下所有内容不迁移（Matt 个人使用）。 |
| **R11 — 排除 in-progress** | `skills/in-progress/` 下所有内容不迁移。 |
| **R12 — 排除 deprecated** | 已废弃 skill 不迁移。 |
| **R13 — 按需排除** | 平台专属 skill（如 `git-guardrails-claude-code`）以及用户明确要求排除的 skill 不迁移。具体排除项需在迁移记录中列出并注明原因。 |

---

## 迁移执行流程

1. **确定源版本**：记录源仓库的 git rev，以便追溯
2. **制定迁移方案**：按 R1–R13 梳理待迁移 skill 列表，确定每个 skill 的目标形态（slash command 或 skill），形成完整迁移方案
3. **用户确认**：将迁移方案展示给用户，由用户确认或调整后，按最终确认的方案执行
4. **执行转换**：按确认的方案逐个 skill 执行转换
5. **逐项记录**：每个 skill 的转换决策和修改内容必须在迁移记录中写清楚
6. **创建迁移记录**：在目标平台的 `migration-logs/` 目录下创建迁移记录文件
7. **更新平台状态**：在 `AGENTS.md` 中反映目标平台的当前状态

---

## 迁移记录要求

每次迁移必须在目标平台的 `migration-logs/` 目录下生成一份迁移记录，使用中文编写。迁移记录分为两种类型：

### 初始迁移记录（`_initial`）

仅用于目标平台**从零到一的首次完整迁移**。后续任何上游变更（新增 skill、更新内容、删除 skill 等）同步均使用增量同步记录格式。内容需详尽，必须包含：

- **源版本信息**：源仓库名、git rev
- **迁移规则**：本次迁移所遵循的规则清单（引用上面的 R1–R13，如有增量规则则在此定义）
- **迁移清单**：逐项列出每个被迁移的 skill/command，包含：
  - 来源路径
  - 类型转换（如 User-invoked → Slash Command）
  - 所做的修改及原因
- **忽略项**：明确列出未迁移的 skill 及其排除原因
- **目标目录结构**：迁移完成后的文件树

### 增量同步记录（`_sync-xxx`）

首次迁移完成后，源 `skills/` 发生任何变更（包括新增 skill、更新内容、删除 skill），将变更同步到目标平台时使用。内容聚焦变更本身：

- **上游变更**：
  - 源 rev：`<old_rev>` → `<new_rev>`
  - 变更概述：简要说明上游发生了什么变化
- **同步清单**：逐项列出每个被同步的变更项，包含：
  - 来源路径及文件
  - 类型转换（如 User-invoked → Slash Command）
  - 所做的修改及原因
- **忽略项**（可选）：上游有变更但决定不同步的项，列出并注明原因
- **目标目录结构**（可选）：仅当同步涉及文件增删时更新

### 迁移记录命名规则

```
YYYY-MM-DD_<short-description>.md
```

- `YYYY-MM-DD`：迁移日期
- `<short-description>`：简短描述（如 `initial`、`sync-tdd-skill`、`update-ref-rules`）

示例：
- `2025-06-16_initial.md` — 首次完整迁移
- `2025-06-20_sync-tdd-skill.md` — 同步 tdd skill 的更新

---

## 同步策略

对 `skills/` 源目录中 skill 的任何修改（新增、更新、删除），必须同步到所有已迁移的目标平台。同步流程：

1. 在源 `skills/` 目录完成修改
2. 按 R1–R13 确定修改对应的转换方式，在目标平台执行对应修改
3. 更新 `AGENTS.md` 中的平台状态信息
4. 在 `migration-logs/` 中创建新的迁移记录
