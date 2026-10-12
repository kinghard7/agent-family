# MIRIX：面向 LLM Agent 的多 Agent 记忆系统

- **英文原标题**：MIRIX: Multi-Agent Memory System for LLM-Based Agents
- **作者/机构**：Yu Wang, Xi Chen（MIRIX AI）
- **arXiv 链接**：https://arxiv.org/abs/2507.07957 （2507.07957v1，2025-07-10）
- **一句话定位**：用六种结构化记忆类型 + 多 Agent 协调框架，取代「扁平单一存储」的传统方案，让 LLM Agent 真正具备长期、个性化、可路由、支持多模态的记忆能力。

---

## 概要

MIRIX 针对现有记忆系统「扁平、范围狭窄、缺乏组合结构、多模态支持差、抽象/扩展能力弱」的痛点，提出一套模块化的多 Agent 记忆系统。它把记忆细分为六种专门类型（Core / Episodic / Semantic / Procedural / Resource / Knowledge Vault），每种由专属的 Memory Manager 管理，并由一个 Meta Memory Manager 负责输入路由与协调，共部署八个 Agent。系统强调两大核心能力——**路由（Routing）** 与 **检索（Retrieving）**，并引入 **主动检索（Active Retrieval）** 机制：先生成话题再据此检索注入。实验上，在自建多模态基准 ScreenshotVQA 上比 RAG 基线准确率高 35% 且存储减少 99.9%；在长对话基准 LOCOMO 上达到 85.4% 的 SOTA。论文还附带一个实时截屏、本地存储的个人助理应用。

---

## 关键信息

### 一、问题动机：现有记忆系统的三大局限

1. **缺乏组合式记忆结构（Lack of compositional structure）**：多数方案把所有历史塞进单一扁平存储，不路由到专门记忆类型，检索低效且不准。
2. **多模态支持差**：以文本为中心的机制在输入多为图像、界面布局、地图等非语言信息时失效。
3. **可扩展性与抽象能力不足**：直接存原始输入（尤其图像）导致存储开销巨大，缺少「只保留关键信息」的抽象层。

对比的现有路线：知识图谱型（Zep、Cognee）擅长结构化实体关系但难表达时序事件/情绪/长文档/多模态；扁平向量型（Letta/MemGPT、Mem0、ChatGPT memory）把对话与偏好简单分块存储，路由能力弱。

### 二、六种记忆类型分类（核心贡献）

| 记忆类型 | 定义与用途 | 关键字段 |
|---|---|---|
| **核心记忆（Core Memory）** | 高优先级、需常驻在 Agent 上下文中的持久信息。借鉴 MemGPT，分 persona（Agent 身份/语气/行为画像）与 human（用户姓名、偏好等长期事实）两块。容量超 90% 时触发受控重写以保持紧凑。 | persona / human 两个 block |
| **情景记忆（Episodic Memory）** | 带时间戳的事件与情境化交互，像结构化日志/日历，用于推理用户作息、时效性、上下文追踪与待办跟进。 | event_type、summary、details、actor（user/assistant）、timestamp |
| **语义记忆（Semantic Memory）** | 与具体时间无关的抽象知识与事实：概念、实体、关系（世界知识 + 用户社交图谱）。除非被概念性覆盖否则持久保留，组织成树状结构。 | name、summary、details、source（user_provided/Wikipedia/inferred） |
| **程序记忆（Procedural Memory）** | 目标导向的流程性知识：how-to 指南、操作工作流、交互脚本（如「如何报差旅费」「如何用 OpenTable 订餐」）。支持任务规划、自动化与目标拆解。 | entry_type（workflow/guide/script）、description、steps（可 JSON 结构化） |
| **资源记忆（Resource Memory）** | 存放用户正在处理、但不属于其它类别的完整/部分文档、转录稿、多模态文件，保障长任务的上下文连续性。 | title、summary、resource_type（doc/markdown/pdf_text/image/voice_transcript）、content |
| **知识库金库（Knowledge Vault）** | 必须逐字精确保存的敏感信息：凭证、地址、联系方式、API key 等。高敏感条目受访问控制保护，排除在日常检索之外防泄漏。 | entry_type（credential/bookmark/contact_info/api_key）、source、sensitivity（low/medium/high）、secret_value |

> 设计要点：每种记忆内部都采用**分层结构**（如 Episodic 有 summary/details，Semantic 按 name/description 组织）。这种异构结构对单 Agent 难以统一管理，因此引出多 Agent 架构。

### 三、系统架构（八个 Agent）

- **Meta Memory Manager（元记忆管理器）**：中枢，负责任务路由——分析输入内容，决定哪些记忆组件相关，分发给对应的 Memory Manager。
- **六个 Memory Manager**：每种记忆类型一个专属管理器，并行更新各自记忆，并在各自类型内去重。
- **Chat Agent（对话 Agent）**：演示如何与拥有记忆访问权的 Agent 交互（记忆系统本身可插拔接入其它 Agent）。
- 合计「六种记忆组件 + 八个 Agent」（六个 Memory Manager + Meta Memory Manager + Chat Agent）。

### 四、记忆存取与路由机制

**主动检索（Active Retrieval）—— 核心机制**
- 解决痛点：传统系统检索需显式触发，否则模型回退到可能过时的参数化知识（例：用户存过「Twitter CEO 是 Linda Yaccarino」，几天后再问却答成 Elon Musk）。反复让用户说「去搜索记忆」不现实。
- 两阶段流程：① Agent 先基于输入上下文**生成一个当前话题（topic）**；② 用该话题从六个记忆组件各检索 Top-10 最相关条目，结果注入 system prompt。检索内容按来源打标签（如 `<episodic_memory>...</episodic_memory>`），让模型同时感知内容与出处。

**多种检索函数**：embedding_match、bm25_match、string_match，并持续扩展；各方法差异化，Agent 按情境自选最合适的。

**记忆更新工作流（Memory Update Workflow）**
1. 收到用户新输入 → 系统先对记忆库做自动检索；
2. 检索结果 + 用户输入 → 交给 Meta Memory Manager 分析、判定相关记忆组件并路由；
3. 对应 Memory Manager **并行更新**各自记忆（同类型内去重）；
4. 更新完成后回报 Meta Memory Manager，后者发确认信号收尾。

**对话检索工作流（Conversational Retrieval Workflow）**
- Chat Agent 收到 query 后先做**粗检索**（横跨六组件，返回高层 summary 而非细节）；
- 再分析 query 决定哪些组件需要**定向精检索**并选检索方法；
- 汇总信息合成最终回答；若 query 涉及更新记忆（新事实/纠错），Chat Agent 可直接与对应 Memory Manager 交互做精确更新。

### 五、关键技术点

- **多模态与抽象**：不存原始图像，只把抽取出的信息存进 SQLite，实现极致存储压缩。
- **流式上传降延迟**：应用每 1.5 秒截屏一次，丢弃高相似帧，累积 20 张唯一截图（约 60 秒）触发更新；逐张即时上传（借助 Gemini 支持 Google Cloud URL 加载图像），端到端延迟从 GPT-4 直传的约 50 秒降到 5 秒以内。
- **混合存储**：Knowledge Vault 等关键信息可本地存储，Resource Memory 等大规模记忆可卸载到云端按需检索（适配可穿戴设备等算力/存储受限场景）。
- **应用形态**：跨平台应用（前端 React-Electron，后端 Uvicorn），实时屏幕监控、树状/列表记忆可视化、本地安全存储保护隐私。论文还展望「可穿戴设备记忆」与「Agent 记忆市场（去中心化记忆资产交易）」两个延伸场景。

### 六、实验与评估结果

**基准一：ScreenshotVQA（自建多模态基准）**
- 数据：3 名博士生一周到一月的高分辨率截屏（5,000–20,000 张/序列，2K–4K 分辨率），人工出题（11/21/55 题）。骨干模型 gemini-2.5-flash-preview-04-17，评测用 GPT-4.1 做 LLM-as-a-Judge。
- 结果（Overall）：MIRIX 准确率 **0.5950**、存储仅 **15.89MB**；Gemini 长上下文基线 0.1166 / 236.70MB；SigLIP@50（RAG）0.4410 / 15.07GB。
- 结论：相比 RAG 基线**准确率 +35%、存储 −99.9%**；相比长上下文基线**准确率 +410%、存储 −93.3%**。

**基准二：LOCOMO（长对话基准，对照 Mem0 设定）**
- 10 段对话，每段约 600 轮、约 26,000 token、约 200 题；题型分 single-hop / multi-hop / temporal / open-domain（排除 adversarial）。骨干模型 gpt-4.1-mini（函数调用能力强于 gpt-4o-mini）。
- 总分（Overall J-score）：MIRIX **85.38%**，优于最强开源基线 LangMem（78.05）超 8 分，接近 Full-Context 上界（87.52）。分项：Single-Hop 85.11、Multi-Hop **83.70**、Temporal 88.39、Open-Domain 65.62。
  - **Multi-Hop 领先最大（超基线 24+ 分）**：因 MIRIX 显式存储「整合后的事件」（如「Caroline 4 年前从家乡瑞典搬来」），无需查询时再拼接碎片信息。
  - **Open-Domain 优势最小**：这类「what if」问题需全局推理，MIRIX 仍依赖 RAG 检索，暴露 RAG 缺乏全局理解的固有瓶颈。
  - Single-Hop 略低于 Full-Context：因问题歧义（如「计划露营」vs「实际露营」），MIRIX 倾向于已确认事件而非早期计划。

---

## 总结

**核心贡献**
1. 系统分析现有 LLM Agent 记忆架构的局限，提出由**六种专门记忆组件 + 八个 Agent** 组成的模块化记忆系统，强调路由与检索两大核心能力。
2. 提出 **Active Retrieval**（先生成话题再检索注入）机制，消除显式触发检索的需求，缓解模型回退到过时参数知识的问题。
3. 引入新的多模态基准 ScreenshotVQA（海量高分辨率截屏），并在该基准与 LOCOMO 上验证 MIRIX 同时取得**更高准确率与极低存储**。
4. 发布可落地的个人助理应用，让记忆能力可被普通用户体验。

**局限**
- Open-Domain 类问题仍受 RAG「缺乏全局理解」瓶颈制约。
- 非推理型骨干模型（gpt-4.1-mini）在需要额外推理步骤时可能失败；系统对模型的函数调用能力依赖较强（每步要调用 Meta Manager 及 0–6 个 Memory Manager）。
- 评测主要依赖 LLM-as-a-Judge，存在问题歧义导致的判分偏差。
- ScreenshotVQA 样本规模较小（仅 3 名用户、共 87 题），泛化性待更大规模真实基准验证。

---

## 对 CodeTeam-MD 的启示

MIRIX 的记忆分类与管理流程对 CodeTeam-MD 的三层长期记忆系统（短期/中期/长期）有直接参考价值。可借鉴点：

### 1. 用「记忆类型」细化三层记忆（最值得借鉴）
不要把长期记忆做成单一扁平存储。可将 MIRIX 六类映射到 Coding Agent 场景：
- **Core Memory → 项目/用户画像**：常驻上下文的高优先信息，对应 CodeTeam-MD 的「用户习惯」与角色 persona（PM/Architect/Dev/Tester/Admin 的稳定行为画像 + 项目约束/技术栈偏好）。借鉴其 persona/human 双块设计与「容量超阈值触发重写」的压缩策略。
- **Episodic Memory → 任务/会话事件日志**：带时间戳记录每次任务、改动、决策与纠错事件（event_type/summary/details/actor/timestamp），用于回溯「上次为什么这么改」、跟进未完成任务，天然契合「纠错闭环」。
- **Semantic Memory → 项目知识图谱**：模块职责、领域概念、API/实体关系、人物（团队成员）等抽象知识，树状组织，支持架构推理。
- **Procedural Memory → 技能库（直接对应「技能自动迭代」）**：把可复用的操作流程（如「如何跑测试」「如何发布」「如何接入某中间件」）以 workflow/guide/script + steps 结构存储。这正是 CodeTeam-MD「技能自动迭代」的落地载体——技能即程序记忆条目，可被检索、复用、版本化。
- **Resource Memory → 工件/文档存储**：需求文档、设计稿、Markdown 规范、PRD、长代码文件摘要等，保障长任务上下文连续。契合「Markdown 驱动」特性。
- **Knowledge Vault → 凭证/敏感配置**：API key、部署凭证、连接串等，带 sensitivity 分级与访问控制，由 Administrator 角色管控，避免在普通检索/对话中泄漏。

### 2. Meta Manager 路由 = Administrator 协调
MIRIX 的 Meta Memory Manager 模式可直接映射到 CodeTeam-MD 的 Administrator 角色：由它分析输入、判定相关记忆类型、把更新/检索任务路由到对应角色或记忆模块，并行处理后汇总确认。这与「角色分工 + 协调」高度同构。

### 3. Active Retrieval：先定话题再检索注入
在每次角色执行任务前，先让 Agent 生成「当前话题」，据此从各记忆类型检索 Top-K 并注入上下文（按来源打标签，如 `<skill>…</skill>`、`<episodic>…</episodic>`）。可避免 Coding Agent 回退到过时假设、重复提问，强化「用户习惯学习」的连续性。

### 4. 差异化检索函数 + 粗检索→精检索两段式
- 提供 embedding_match / bm25_match / string_match 等多种检索方式，让 Agent 按场景自选（代码精确匹配用 string/bm25，语义查找用 embedding）。
- 对话/任务先做跨类型粗检索（只返回 summary），再对相关类型定向精检索——降低 token 开销，适配 CLI Coding Agent 的上下文预算。

### 5. 抽象而非存原文 + 去重
借鉴「不存原始大文件、只存抽取后的结构化信息」与「同类型内去重」，控制 Markdown 记忆文件膨胀，提升长期可维护性。纠错信息应整合为「已确认结论」存入（如 MIRIX 把碎片拼成整合事件），便于多跳推理与跨任务复用。

### 6. 纠错闭环的落地路径
MIRIX 中「用户提供纠正 → Chat Agent 直接与对应 Memory Manager 交互做精确更新」的模式，可作为 CodeTeam-MD 纠错闭环的实现范式：用户纠错 → 定位相关记忆类型（多为 Procedural 技能或 Core 习惯）→ 精确更新对应 Markdown 条目 → 后续任务自动生效。
