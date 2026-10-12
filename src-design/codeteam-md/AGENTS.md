# AGENTS.md — CodeTeam-MD 团队规则（宿主无关）

本文件定义团队的协作规范，**跨宿主通用**（Claude Code / Codex / CodeBuddy 均读它）。它是真身；`.claude/agents/` 下的文件只是 Claude Code 的接入适配器。

## 1. 角色与职责

| 角色 | 负责状态 | 用户可见 | 职责定义 |
|------|---------|---------|---------|
| ProductManager | draft / pending_review / blocked | ✅ 唯一入口 | @ROLES/ProductManager/role.md |
| Architect | pending_arch | ❌ | @ROLES/Architect/role.md |
| FullstackDev | pending_dev | ❌ | @ROLES/FullstackDev/role.md |
| Tester | pending_test | ❌ | @ROLES/Tester/role.md |
| Administrator | 幕后（不占状态） | ❌ | @ROLES/Administrator/role.md |

## 2. 通讯协议（共享 Markdown 黑板 + 拉取）

Agent 之间**不直接对话**，通过共享 Markdown 文件通讯。每个 Agent 被委派后，扫描 `SHARED/TASKS/`，认领 `owner == 自己` 的任务卡。

### 2.1 任务卡三文件结构

每个任务一个目录 `SHARED/TASKS/TASK-xxx/`：

| 文件 | 作用 | 加载时机 |
|------|------|---------|
| `meta.md` | frontmatter 状态机（status/owner/from/updated）+ append-only 交接记录 | 扫描时读 |
| `summary.md` | 渐进式摘要，**仅实质进展时追加**（append-only） | 唤醒时读 |
| `detail.md` | 完整需求、方案、交接记录 | 按需读 |

`meta.md` 模板：

```markdown
---
id: TASK-001
status: pending_dev
owner: FullstackDev
from: Architect
updated: 2026-06-20
---
## 交接记录（append-only，不删改）
- [2026-06-20 Architect] 方案已设计，详见 detail.md
- [2026-06-20 FullstackDev] 接手开发
```

### 2.2 全局任务索引

`SHARED/TASKS/INDEX.md` 常驻加载，每条一行带关键词，用于唤醒历史任务。

### 2.3 防冲突

- 交接记录 **append-only**：只追加不修改，防写冲突、可回溯。
- **单 owner 串行**：同一时刻一个任务只有一个 owner。
- **Administrator 是观察者**：直接读 TASKS/ 与日志，无需各角色抄送。

## 3. 工作流状态机

```
draft(PM) → pending_arch(Architect) → pending_dev(Dev) → pending_test(Tester) → pending_review(PM终审) → done
```

| 状态 | owner | 含义 |
|------|-------|------|
| `draft` | PM | 需求收集中，尚未拆任务 |
| `pending_arch` | Architect | 待出技术方案 |
| `pending_dev` | FullstackDev | 待开发 |
| `pending_test` | Tester | 待测试 |
| `pending_review` | PM | 待 PM 终审（交付前质控） |
| `done` | — | 完成 |
| `rejected` | 对应上游 | 打回标记（带原因），回到根因所在上游 |
| `blocked` | PM | 卡住，需用户决策 |

### 流转规则

1. **PM 是唯一出入口**：draft 进、pending_review 出，中间自动流转，用户只见 PM。
2. **打回到任意上游（Gate）**：下游发现上游问题 → 置 `rejected` + 写明原因 → 回到**根因所在**上游（如 Tester 发现是架构问题，直接打回 `pending_arch`）。
3. **blocked 触发用户介入**：Agent 无法解决（需求不明/外部依赖）→ 置 `blocked` → PM 回头问用户。这是除「完成」外唯一打断用户的情况。
4. **每次状态变更**：① `summary.md` 追加实质进展；② 通知 Administrator。

## 4. 记忆系统（三层）

- 常驻层 `SHARED/MEMORY/INDEX.md` + `core/`：启动即加载（项目概览、用户习惯）。
- 召回层 `SHARED/MEMORY/recall/`：凭 INDEX 关键词按需唤醒（决策日志、纠错案例，append-only）。
- 归档层 `SHARED/MEMORY/archive/`：仅显式追溯时读。

记忆由 **Administrator 专职写入**，由实质进展事件触发，避免碎片化。

## 5. 纠错与进化（L1/L2/L3）

PM 判断用户反馈意图，分三级：
- **L1 内容纠正**：改本次产出，不进纠错库。
- **L2 偏好信号**：静默更新 `core/user_preferences.md` + 记 `recall/correction_cases.md`。
- **L3 系统问题**：记纠错库 + Administrator 生成技能 diff 草案 → **用户确认才落地**（human-in-the-loop）。

## 6. 技能系统

技能 = 文件夹 + `SKILL.md`（frontmatter 仅 `name` + `description`）。三级作用域：
- 全局 `SHARED/SKILLS/`
- 角色私有 `ROLES/<角色>/skills/`
- 外部导入 `EXTERNAL_SKILLS/`（须 Administrator 审核后登记进 INDEX 才生效）

加载策略：启动只读 `SKILLS/INDEX.md` 中各技能 description，命中才读 `SKILL.md` 全文。

## 7. 风格指南

- 中文沟通；产出文档简洁、最小复杂度（用户核心偏好，见 user_preferences.md）。
- 默认最简方案，不过度设计。
