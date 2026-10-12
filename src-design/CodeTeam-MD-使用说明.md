# CodeTeam-MD 使用说明

CodeTeam-MD 是一套 **Markdown 原生、宿主无关的 Agent 团队编排层**。它本身不是 Agent，而是一组协议文件（角色定义 / 共享记忆 / 任务状态机），让现成的 CLI Agent（Claude Code / Codex / CodeBuddy）按统一规则像一个研发团队那样协作。

你全程只跟一个入口——**ProductManager（PM）**——对话。架构师、开发、测试、管理员四个角色在背后由 PM 通过 Task 工具调度，你对他们无感知。

---

## 一、核心概念（先看这张表）

| 角色 | 持有状态 | 你可见 | 干什么 |
|------|---------|:----:|------|
| ProductManager | `draft` / `pending_review` / `blocked` | ✅ 唯一入口 | 需求澄清、拆任务、终审、收反馈 |
| Architect | `pending_arch` | ❌ | 出技术方案、拆 Epic→Task |
| FullstackDev | `pending_dev` | ❌ | 写代码、自测 |
| Tester | `pending_test` | ❌ | 测试、出 Bug 报告 |
| Administrator | 幕后（不占状态） | ❌ | 记忆管理、纠错进化 |

**工作流状态机：**

```
draft(PM) → pending_arch → pending_dev → pending_test → pending_review(PM终审) → done
                  ↑__________ rejected 打回根因上游 __________↓
任意环节卡住 → blocked → PM 回头问你
```

> 你只在两种情况被打断：① 任务 `blocked`（需求不明/外部依赖）；② 交付 `pending_review` 完成。中间四个角色全自动流转。

---

## 二、目录结构

```
codeteam-md/
├── CLAUDE.md              # 宿主入口：启动即加载身份(=PM)+规则+常驻记忆
├── AGENTS.md              # 团队规则真身（跨宿主通用）
├── .claude/
│   ├── agents/            # Claude Code 的 subagent 接入适配器
│   │   ├── architect.md
│   │   ├── fullstack-dev.md
│   │   ├── tester.md
│   │   └── administrator.md
│   └── commands/
│       └── codeteam.md    # /codeteam 命令
├── ROLES/                 # 五个角色的完整职责定义
│   ├── ProductManager/role.md
│   ├── Architect/role.md
│   ├── FullstackDev/role.md
│   ├── Tester/role.md
│   └── Administrator/role.md
└── SHARED/                # 团队共享黑板
    ├── TASKS/
    │   ├── INDEX.md       # 任务索引（常驻加载）
    │   └── TASK-xxx/      # 每个任务一目录：meta.md / summary.md / detail.md
    ├── MEMORY/
    │   ├── INDEX.md       # 记忆索引（常驻加载）
    │   ├── core/          # 启动即加载：项目概览、用户偏好
    │   ├── recall/        # 按关键词召回：决策日志、纠错案例
    │   └── archive/       # 仅显式追溯时读
    └── SKILLS/
        ├── INDEX.md       # 技能索引
        └── <skill>/SKILL.md
```

**关键机制：** 各角色之间不直接对话，全靠 `SHARED/TASKS/TASK-xxx/` 下的三个 Markdown 文件通讯——

| 文件 | 作用 |
|------|------|
| `meta.md` | frontmatter 状态机（status/owner/from/updated）+ append-only 交接记录 |
| `summary.md` | 渐进式摘要，仅实质进展时追加 |
| `detail.md` | 完整需求、技术方案、Bug 报告 |

---

## 三、启动方式（通用，必读）

CodeTeam-MD 靠 **当前工作目录下的 `CLAUDE.md`** 来激活身份与团队规则。所以**必须在含有这套骨架的目录里启动 Claude Code**：

```bash
cd <含 CLAUDE.md/AGENTS.md/ROLES/SHARED 的目录>
claude
```

> ⚠️ 如果在上一层目录启动，不会加载到这套 CLAUDE.md，团队骨架就是“没激活”的。

启动后你就是 PM。两种用法：

| 用法 | 效果 |
|------|------|
| 直接用中文说需求 | 当作新需求：建任务卡 → 澄清 → 自动流转 → 终审交付 |
| `/codeteam`（空参） | 展示任务看板 + 团队状态摘要 |
| `/codeteam <需求描述>` | 作为新需求处理，建 `TASK-xxx` 并开始澄清 |

---

## 四、全新项目怎么用

最简单——直接把这套骨架当成新项目的根，代码和团队骨架长在一起。

**步骤：**

1. **复制骨架到新项目根目录**（或直接把 codeteam-md 改名当新项目根）：

   ```bash
   cp -r codeteam-md/ ~/projects/my-new-app
   cd ~/projects/my-new-app
   ```

2. **填初始记忆**（可选但强烈建议）——编辑 `SHARED/MEMORY/core/`：
   - `project_summary.md`：技术栈、约束、范围
   - `user_preferences.md`：你的习惯（命名风格、是否要测试、文档详略等）

   PM 启动时会常驻加载这两份，越早写越省后续澄清。

3. **启动并提需求：**

   ```bash
   claude
   ```
   然后直接说，例如：「做一个待办清单 App，前端 React，后端 Node + SQLite」。

4. PM 会建 `TASK-001`、澄清几个关键点、然后自动流转给架构→开发→测试，最后回到你这里终审。**代码会直接写进这个项目目录**。

**适用：** 从零开始、希望团队记忆和代码同仓管理、长期演进的项目。

---

## 五、老项目怎么用（两种方式）

老项目已有自己的代码和目录结构，接入方式取决于你是否愿意把骨架文件放进老仓库。

### 方式 A：拷骨架进老项目（一体化）

把团队骨架直接复制到老项目根目录，骨架与老代码同仓。

**步骤：**

1. 复制骨架文件进老项目根（注意别覆盖老项目已有的同名文件）：

   ```bash
   cd ~/projects/legacy-app
   cp -r <codeteam-md>/ROLES   ./ROLES
   cp -r <codeteam-md>/SHARED  ./SHARED
   cp -r <codeteam-md>/.claude ./.claude
   cp <codeteam-md>/AGENTS.md  ./AGENTS.md
   # CLAUDE.md：若老项目已有，把 codeteam-md/CLAUDE.md 的内容并进去；没有则直接拷
   ```

2. 在 `SHARED/MEMORY/core/project_summary.md` 写清老项目的技术栈、目录约定、不可碰的部分。

3. `claude` 启动，正常提需求。FullstackDev 会**就地修改老项目代码**。

**优点：** 团队对老代码有完整上下文，改动直接落在仓库里，记忆与代码同源、可随仓库版本化。
**代价：** 往老仓库里塞了若干新文件/目录（`ROLES/` `SHARED/` `.claude/` `AGENTS.md`），需要团队接受；`.gitignore` 可能要调整。

**适用：** 你能掌控老仓库、希望长期用这套流程、能接受多几个目录的项目。

---

### 方式 B：骨架与代码分离（外挂）

骨架单独放一个目录，老项目代码原地不动；只把老项目路径告诉 PM，让开发写到外部路径。

**步骤：**

1. 骨架独立放置，不动老项目：

   ```bash
   ~/codeteam-md/          # 团队骨架（在这里启动 claude）
   ~/projects/legacy-app/  # 老项目，保持原样
   ```

2. 在 `SHARED/MEMORY/core/project_summary.md` 里登记老项目**绝对路径**和约束，例如：

   ```markdown
   ## 目标代码库
   - 路径: /Users/you/projects/legacy-app
   - 技术栈: Spring Boot 2.7 + MySQL
   - 约束: 不改 com.legacy.core 包；新代码放 com.legacy.feature
   ```

3. `cd ~/codeteam-md && claude`，提需求时明确目标路径。FullstackDev 会**读写老项目的外部路径**，骨架目录只存任务卡/记忆/方案，不混入老代码。

**优点：** 老仓库零侵入、git 历史干净；一套骨架可同时挂多个老项目。
**代价：** 骨架与代码两地分离，团队需要靠 `project_summary.md` 里的路径建立上下文；启动时要确保对老项目路径有读写权限。

**适用：** 不方便改动老仓库、需要零侵入、或一套团队服务多个项目的场景。

---

### 老项目两种方式对比

| 维度 | 方式 A：拷骨架进老项目 | 方式 B：骨架与代码分离 |
|------|----------------------|----------------------|
| 老仓库侵入 | 多几个目录/文件 | 零侵入 |
| 上下文获取 | 团队天然在仓库里 | 靠 `project_summary.md` 登记路径 |
| 记忆与代码 | 同仓、可一起版本化 | 分离，骨架独立维护 |
| 一套骨架挂多项目 | 不方便 | 方便 |
| 推荐场景 | 长期专用、可掌控仓库 | 零侵入、多项目复用 |

---

## 六、日常协作小贴士

- **你只跟 PM 说话**：不用关心谁在干活，描述需求和验收标准即可。
- **被问 `blocked` 时**：说明这是需求不明或外部依赖，PM 拿到你的答复后会自动解除继续。
- **反馈会被自动分级**（你不用手动分）：
  - **L1 内容纠正**（「这个函数名改成 camelCase」）→ 当场改，不进库。
  - **L2 偏好信号**（「我说过好几次了，别上复杂方案」）→ 静默写进 `user_preferences.md`，以后自动遵守。
  - **L3 系统缺陷**（「你又犯这个错」）→ Administrator 生成技能改进草案，**经你确认才落地**。
- **看进度**：随时 `/codeteam` 查任务看板。
- **越早填 `core/` 记忆，后续澄清越少**：把技术栈、命名习惯、最简方案偏好写进去，团队全程遵守。

---

## 七、常见问题

**Q：启动后好像没有“团队”的样子？**
A：八成是启动目录不对。确认 `pwd` 下能看到 `CLAUDE.md` 和 `AGENTS.md`，否则 `cd` 进骨架目录重启。

**Q：可以用 Codex / CodeBuddy 吗？**
A：可以。`AGENTS.md` 是跨宿主通用的真身规则，`.claude/agents/` 只是 Claude Code 的接入适配器。其它宿主按各自的 subagent/命令机制接入同一套 `ROLES/` 和 `SHARED/` 即可。

**Q：任务卡能手动看吗？**
A：能。`SHARED/TASKS/TASK-xxx/` 下三个 md 都是纯文本，随时可读，交接记录 append-only 可完整回溯。
