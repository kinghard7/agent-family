# CodeTeam-MD 开发计划

**关联设计**：`design/CodeTeam-MD.md`（v1.1）
**代码位置**：`codeteam-md/`
**当前阶段**：第一阶段 — 纯 Markdown 骨架，在 Claude Code 单宿主跑通

---

## 1. 阶段目标

把设计文档落地为可在 Claude Code 上运行的真实项目。第一阶段只做**纯 Markdown 骨架**：不写编排脚本（`orchestrator.py`）、不做 Web 管理面板，先跑通完整流水线（PM→架构→开发→测试→终审）及打回 / blocked / 纠错三条分支。

## 2. 两个关键实现选择（已确认）

1. **自定义目录为真身**：`ROLES/`、`SHARED/` 是唯一数据源；`.claude/agents/*.md` 是薄适配器，仅 frontmatter + `@` 引用指向真身。好处：跨宿主可移植（Codex/CodeBuddy 直接读 ROLES/），Claude Code 通过适配器接入。
2. **subagent 委派驱动角色**：PM = 主会话（用户唯一入口，由 `CLAUDE.md` 注入身份）；Architect/Dev/Tester/Administrator = `.claude/agents/` 下的 subagent；PM 通过 Task 工具委派，按状态机流转任务卡。

## 3. Claude Code 真实格式（已验证，务必遵守）

| 构件 | 路径 | frontmatter | 调用方式 |
|------|------|-------------|---------|
| Subagent | `.claude/agents/<name>.md` 单文件 | `name` / `description` / `tools`（逗号分隔）/ `color` | Task 工具委派（**无 /invoke**） |
| Skill | `<name>/SKILL.md` | **仅** `name` + `description` | 渐进式披露 |
| Command | `.claude/commands/<name>.md` | 正文即 prompt | `$ARGUMENTS` 传参 |
| CLAUDE.md | 项目根 | — | always-loaded，`@path` 引用 |

## 4. 实际目录结构（已落地）

```
codeteam-md/
├─ CLAUDE.md                      # 宿主入口，注入 PM 身份 + @AGENTS.md + 常驻 INDEX
├─ AGENTS.md                      # 团队规则/通讯协议/状态机（真身，跨宿主）
├─ ROLES/                         # 真身：角色定义
│  ├─ ProductManager/role.md
│  ├─ Architect/role.md
│  ├─ FullstackDev/role.md
│  ├─ Tester/role.md
│  └─ Administrator/role.md (+ reports/)
├─ SHARED/                        # 真身：记忆/任务/技能
│  ├─ MEMORY/{INDEX.md, core/, recall/, archive/}
│  ├─ SKILLS/{INDEX.md, markdown-spec/SKILL.md}
│  └─ TASKS/INDEX.md
├─ .claude/
│  ├─ agents/{architect,fullstack-dev,tester,administrator}.md   # 适配器
│  └─ commands/codeteam.md        # 看板/启动命令
└─ .gitignore                     # 个性化隔离（设计 §11.2）
```

PM 不做 subagent —— 它就是主会话，身份经 `CLAUDE.md` 注入。

## 5. 文件职责对照设计章节

| 文件 | 对应设计 |
|------|---------|
| `CLAUDE.md` | §1 定位 + §3.1 PM + 入口约定 |
| `AGENTS.md` | §6 工作流 + §7 通讯 + §4/§5/§8 摘要 |
| `ROLES/*/role.md` | §3 各角色 |
| `.claude/agents/*.md` | Claude Code 接入适配器 |
| `SHARED/MEMORY/*` | §4 三层记忆 |
| `SHARED/TASKS/INDEX.md` | §7.2 任务索引 |
| `SHARED/SKILLS/*` | §8 技能系统 |
| `.gitignore` | §11.2 个性化隔离 |

## 6. 验证清单（端到端跑通）

在 `codeteam-md/` 下用 Claude Code 实测：

1. **注入**：启动会话，确认 CLAUDE.md 正确注入 PM 身份 + 两个 INDEX。
2. **正常路径**：提需求「加个用户登录功能」→ PM 建 `TASK-001/` → draft → 委派 architect（pending_arch）→ pending_dev → pending_test → pending_review → done；每步 summary.md 追加、meta.md 交接日志 append。
3. **打回路径**：Tester 发现存储方案不合规 → rejected 直接打回 pending_arch。
4. **blocked 路径**：Architect 遇「是否支持第三方 OAuth」不明 → blocked → PM 回头问用户。
5. **纠错路径（L3）**：用户说「又做复杂了」→ PM 判 L3 → 委派 administrator 写 correction_cases.md + 生成技能 diff 草案 → 呈现用户确认。

跑通即第一阶段完成。

## 7. 后续阶段（不在本期）

- 可选轻量 `orchestrator.py`（任务量增大时批量扫描/唤起，不引重型框架）。
- 可选 Web 管理面板（§9.2，单文件服务 + 静态页，单一数据源）。
- 跨宿主验证（Codex / CodeBuddy）。
