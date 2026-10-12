# AI Agent 记忆综述

> **英文原标题**：Memory in the Age of AI Agents: A Survey (Forms, Functions and Dynamics)
> **arXiv**：https://arxiv.org/abs/2512.13564 （v2, 2026-01-13；约 107 页）
> **机构**：新加坡国立、人大、复旦、北大、南洋理工、同济、UCSD、港科广、Griffith、佐治亚理工、OPPO、牛津等
> **配套资源**：论文清单 GitHub <https://github.com/Shichun-Liu/Agent-Memory-Paper-List>
> **一句话定位**：迄今最系统的「Agent 记忆」综述，提出 **Forms（形式）—Functions（功能）—Dynamics（动态）** 三维统一框架，把碎片化的记忆研究整合为可复用的概念蓝图与工程参考。

---

## 概要

记忆（Memory）是基础模型驱动的 Agent 实现长程推理、持续适应、复杂环境交互的核心能力，但该领域因术语滥用（episodic/semantic/parametric 等）和动机分散而高度碎片化，传统的「长期/短期记忆」二分法已不足以刻画现状。本综述首先界定 Agent 记忆的范围，并与 LLM 记忆、检索增强生成（RAG, Retrieval-Augmented Generation）、上下文工程（Context Engineering）严格区分；随后用 **形式—功能—动态** 三个统一视角重构整个领域：形式回答「什么承载记忆」（token 级 / 参数化 / 潜在），功能回答「为何需要记忆」（事实 / 经验 / 工作记忆），动态回答「记忆如何运作与演化」（形成—演化—检索的生命周期）。最后给出 benchmark 与开源框架清单，并展望生成式记忆、自动化管理、RL 内化、多模态、多 Agent 共享记忆、可信记忆等八大前沿。核心主张：记忆不是附属存储，而应被视为未来智能体设计中的**一等公民（first-class primitive）**。

---

## 关键信息

### 一、Agent 记忆的形式化定义与边界

论文用统一的读/写算子建模记忆系统。记忆状态 `M_t` 不限定内部结构（可为文本缓冲、键值存储、向量库、图或混合），并定义三大生命周期算子（见第三节）。关键洞见：**短期 vs 长期记忆并非架构上的独立模块，而是由「形成/演化/检索」三算子被调用的时间模式（temporal pattern）涌现出来的**——任务内（inside-trial）累积即短期，跨任务（cross-trial）持久即长期。

**与相邻概念的辨析（重要）**：
- **vs. LLM Memory**：Agent 记忆几乎包含早期所谓「LLM 记忆」（如 MemoryBank、MemGPT）。但纯模型内部机制（KV cache 管理、长上下文架构如 Mamba/RWKV、注意力稀疏化）属于 LLM 记忆，不在 Agent 记忆范畴——它们扩展的是模型表征容量，不支持跨任务持久化与主动记忆操作。
- **vs. RAG**：两者共享向量索引、语义检索等技术栈。区别在场景：RAG 主要为单次推理注入**静态外部知识**（HotpotQA 等知识密集任务）；Agent 记忆在**持续交互**中不断把自身行动与环境反馈写入**可演化的持久记忆库**（LoCoMo、LongMemEval、GAIA、SWE-bench 等）。边界随 Agentic RAG 出现而模糊。
- **vs. Context Engineering**：上下文工程把上下文窗口视为**受限计算资源**做调度优化（资源管理范式）；Agent 记忆关注**持久演化的认知状态**（认知建模范式）。前者是「实现层」，后者是「认知基质」。二者在工作记忆/滚动摘要等技术上重叠。

### 二、Forms（形式）：什么承载记忆？

三大形式：
1. **Token 级记忆（Token-level）**：以显式离散单元存储，外部可见、可编辑、可解释，是最主流、工作最多的形式。按拓扑组织程度细分为：
   - **Flat 扁平（1D）**：无拓扑，序列/集合累积（chunk、对话、轨迹、向量条目）。优点简单可扩展，缺点缺乏关系组织、随规模增长易冗余噪声。代表：MemGPT、Mem0、MemoryBank、Reflexion。
   - **Planar 平面（2D）**：单层显式拓扑（图/树/表）。代表：A-Mem、Zep、HuaTuo、M3-Agent、MemTree、HippoRAG。
   - **Hierarchical 层级（3D）**：多层级 + 跨层链接，支持纵向抽象与跨层推理。代表：GraphRAG、G-Memory、HiAgent、HippoRAG2。
2. **参数化记忆（Parametric）**：信息编码进模型权重，前向计算时隐式访问。分内部/外部参数化。
3. **潜在记忆（Latent）**：存于隐藏状态/连续表征/演化潜结构。介于显式数据与固定权重之间，平衡灵活性与效率；可读性低 → 天然隐私保护；高表达力 → 适合多模态、边缘部署、加密敏感场景。牺牲可解释性换高密度与压缩率。

### 三、Functions（功能）：为何需要记忆？

突破粗糙的时间二分，提出三大功能支柱（映射到「编码—处理—检索」认知循环）：
1. **事实记忆（Factual Memory）**——「Agent 知道什么」。声明性知识库，借鉴认知科学的 episodic（情景）/ semantic（语义）记忆，在 Agent 中表现为**处理连续谱**：原始情景轨迹 → 摘要/反思/实体抽取/事实归纳 → 语义事实库。保证三特性：一致性（consistency）、连贯性（coherence）、适应性（adaptability）。分**用户事实记忆**（身份、稳定偏好、约束、历史承诺）与**环境事实记忆**（文档状态、资源、其他 Agent 能力）。
2. **经验记忆（Experiential Memory）**——「Agent 如何改进」。从过往轨迹/成败中抽象出程序性与策略性知识，是持续学习与自进化的关键。按抽象层级分三类（对 CodeTeam-MD 高度相关）：
   - **Case-based（案例型）**：最小加工的原始记录，高保真、可回放/作 in-context 示例（轨迹、解法）。代表：Memento、ExpeL、Synapse、MapCoder。
   - **Strategy-based（策略型）**：抽象出「如何做」的可迁移知识，再分三层：原子 **Insights**（决策规则/反思启发）、序列 **Workflows**（可执行例程，如 AWM、Agent KB）、模式 **Patterns**（认知模板，如 Buffer of Thoughts、ReasoningBank）。是可编辑、可审计、可组合的高层脚手架。
   - **Skill-based（技能型）**：把抽象策略操作化为可验证动作——Code Snippets（如 Voyager 不断增长的技能库、Darwin Gödel Machine 自改写代码）、Functions/Scripts、APIs、MCP（Model Context Protocol，统一工具发现与调用、按需加载降低上下文开销）。三准则：可被调用、结果可验证、可与其他技能组合。
   - **Hybrid（混合）**：高级架构同时整合案例（接地证据）+ 策略（泛化逻辑）+ 技能（执行），动态选格式。
3. **工作记忆（Working Memory）**——「Agent 当前在想什么」。容量受限、动态控制的临时草稿区，单轮（输入压缩、观察抽象）与多轮（状态固化、认知规划、层级折叠如 Context-folding/AgentFold/DeepAgent）。

### 四、Dynamics（动态）：记忆如何运作与演化？——记忆生命周期（重点）

三大基础过程构成相互反馈的闭环：**形成 → 演化 → 检索 →（推理与环境反馈回流）→ 形成**。

**1. Memory Formation 记忆形成**（「如何抽取记忆」）——把原始、噪声、冗余的上下文压缩为信息密集的知识单元，五类操作：
- **Semantic Summarization 语义摘要**：长数据 → 紧凑高层摘要（增量式 / 分区式）。
- **Knowledge Distillation 知识蒸馏**：抽取具体认知资产（事实型 / 经验型 insight）。
- **Structured Construction 结构化构建**：组织成知识图谱/层级树，提升可解释性与多跳推理。分实体级（三元组抽取，如 Mem0g、GraphRAG、Zep 三层时序图）与块级（如 MemTree 自底向上、H-MEM 自顶向下四层、A-MEM 网状笔记、G-Memory 三图）。
- **Latent Representation 潜在表征**：直接编码为向量/KV 态（如 MemoryLLM、M+、MemGen），跨模态统一、密度高但黑箱。
- **Parametric Internalization 参数内化**：经梯度把外部记忆固化进权重。分**知识内化**（模型编辑 MEND/ROME/MEMIT、LoRA 适配器）与**能力内化**（SFT/DPO/GRPO 学习推理轨迹）。零延迟访问，但有灾难性遗忘与高更新成本、难精确修改。

> 强调：五类操作并非互斥，单一算法可组合多种并跨表征转换。

**2. Memory Evolution 记忆演化**（「如何精炼记忆」）——把新记忆整合进既有库，三机制：
- **Consolidation 巩固**：合并新旧、反思整合形成更泛化 insight（局部合并 / 簇级融合 / 全局整合），使学习累积而非孤立。
- **Updating 更新**：解决新旧冲突、纠错补充，适应环境/任务变化（外部库冲突消解 + 内部模型参数更新）。
- **Forgetting 遗忘**：剪除过时冗余。三轴：**时间型**（MemGPT 溢出淘汰、衰减权重软遗忘）、**频率型**（LFU/LRU，如 MemOS、KARMA）、**重要性驱动**（融合时间+频率+语义+情感显著性，LLM 判断价值）。提醒：LRU 等启发式可能误删长尾关键知识。
- 前沿：MemEvolve 提出元进化框架，让**记忆架构本身**随经验自学习自适应。

**3. Memory Retrieval 记忆检索**（「如何利用记忆」）——四步流水线：
- **Timing & Intent 时机与意图**：何时触发、查哪个库。从被动指令触发 → 自主自调节（快慢思考、潜在可训练触发如 MemGen；分层路由如 H-MEM coarse-to-fine）。风险：Agent 高估自身知识不触发检索 → 静默失败/幻觉。
- **Query Construction 查询构建**：把原始输入翻译为对齐记忆索引的检索信号（查询分解 / 查询改写，如 HyDE、PRIME 的 Planner Agent）。
- **Retrieval Strategies 检索策略**：词法（lexical）/ 语义（dense embedding）/ 图遍历（graph）/ 混合（hybrid）。
- **Post-Retrieval 检索后处理**：重排、过滤、聚合压缩，保证最终上下文简洁连贯。

### 五、多 Agent 共享记忆（Shared Memory in MAS）——对本框架最关键

- **演进脉络**：① 早期**孤立局部记忆 + 显式消息传递**（如 MetaGPT、ChatDev 通过对话历史/通信协议交换信息）——避免干扰但冗余、上下文碎片、通信开销随团队规模剧增；② **集中式共享记忆**（全局向量库、黑板系统 blackboard、共享文档）——支撑联合注意力、减少重复、便于长程协调、角色交接与共识构建；但带来记忆杂乱、写竞争（write contention）、缺乏角色/权限感知的访问控制。
- **未来方向**：
  - **Agent 感知的共享记忆**：读写行为按角色、专长、信任度（role/expertise/trust）条件化，实现结构化可靠的知识聚合。
  - **学习驱动的共享记忆管理**：不靠手工同步/摘要/冲突消解策略，而训练 Agent 基于长程团队表现决定「何时、贡献什么、如何贡献」。
  - **潜在记忆作为跨异构信号的统一基质**（论文认为 latent memory 是有前景路径）。
- 相关系统：G-Memory（三层图、为每个 Agent 提供不同粒度定制记忆）、Memory Sharing、GameGPT、Intrinsic Memory Agents（私有 + 共享读写）、LEGOMem、RCR-Router（多 Agent 间预算感知 token 分配）。

### 六、评估方法与资源

- **记忆/终身/自进化 benchmark**：LoCoMo、LongMemEval（长对话）；GAIA、xBench、BrowseComp（深研究/搜索）；SWE-bench Verified（代码修复）；StreamBench（终身学习）；MemoryAgentBench、Evo-Memory、MemoryBench（自进化/持续学习）。
- **隐含相关 benchmark**：ALFWorld/ScienceWorld/BabyAI（具身）、WebShop/WebArena/ToolBench（Web 与工具）、AgentGym/AgentBoard（多任务）。
- **开源框架**（按是否支持事实/经验/多模态记忆与结构对比）：MemGPT（层级 S/LTM）、Mem0（图+向量）、MIRIX（结构化、多模态）、MemoryOS（S/M/LTM 分层）、MemOS（树记忆+memcube）、Zep（时序知识图）、LangMem、Cognee、Memary、Memori、ReMe、MineContext（上下文工程）等。多数用向量/结构化存储支持事实记忆，越来越多建模经验轨迹，多模态新近涌现。

### 七、开放挑战与八大前沿

1. **检索 → 生成（Memory Generation）**：从查询静态库转向按需主动合成记忆表征（retrieve-then-generate 如 ComoRAG/G-Memory；direct generation 如 MemGen/VisMem）。未来应：上下文自适应、跨异构信号融合、可学习自优化。
2. **自动化记忆管理**：从手工规则 → Agent 自主管理。把记忆增删改查作为**显式工具调用**纳入 Agent 决策环，使其推理自身记忆操作；自组织、自演化、层级自适应架构。
3. **RL 内化记忆能力**：RL-free（启发式/prompt 驱动）→ RL-assisted（RL 治理部分环节，如 RMM 重排、Mem-α 写入、Memory-R1 管理、Context Folding 工作记忆）→ 全 RL 驱动（端到端学习记忆架构与控制策略，减少人类先验，覆盖全生命周期）。
4. **多模态记忆**：视觉/视频较成熟，音频等欠探索；目标是真正的 omnimodal 统一表征与时序一致。
5. **多 Agent 共享记忆**（见第五节）。
6. **记忆服务世界模型（World Model）**：从数据缓存 → 状态模拟；双系统架构（快 SSM + 慢 VLM/显式库）、主动记忆策略（认知工作区主动整理/摘要/丢弃）。
7. **可信记忆（Trustworthy Memory）**：三支柱——**隐私**（细粒度权限、用户掌控保留策略、加密/端侧/联邦、差分隐私、可验证遗忘）、**可解释性**（可追溯访问路径、记忆影响因果图、反事实推理、调试工具）、**抗幻觉**（冲突检测、低置信弃权、回退模型先验、多 Agent 交叉校验、机制可解释性）。愿景：OS 式抽象——分段、版本控制、可审计、Agent 与用户共管。
8. **人类认知联系**：现有架构对应 Atkinson-Shiffrin 多存储模型与 Tulving 的 episodic/semantic/procedural 分类；但人脑是**建构性重构**而非逐字回放。未来引入类睡眠的**离线巩固（offline consolidation）**与生成式回放（基于 CLS 互补学习系统理论），周期性把情景流压缩为参数化直觉，解决稳定—可塑性困境。

---

## 总结

**核心观点**：记忆是把静态 LLM 转化为可持续适应 Agent 的基石，应作为一等公民来设计。本综述用 **Forms（token级/参数化/潜在）— Functions（事实/经验/工作）— Dynamics（形成/演化/检索生命周期）** 三维框架统一了碎片化领域，并明确：短期/长期之分是时间调用模式的涌现而非架构模块。

**未来方向**：领域正经历范式转移——① 从检索中心转向**生成式记忆**；② 从手工工程转向**自动化/自组织**管理；③ 从启发式流水线转向 **RL 端到端内化**；④ 向多模态、多 Agent 共享、可信、类脑离线巩固扩展。终极图景是记忆成为与推理决策协同进化的**完全可学习、自适应、自组织子系统**。

---

## 对 CodeTeam-MD 的启示

CodeTeam-MD 是运行在 CLI Coding Agent 之上、Markdown 驱动、角色分工、三层长期记忆、习惯学习与纠错闭环、技能自动迭代的多 Agent 框架。本综述可直接作为其记忆设计蓝图：

### 1. 用「生命周期三算子」统一记忆操作（最核心借鉴）
把 CodeTeam-MD 的记忆读写显式抽象为 **形成（Formation）/ 演化（Evolution）/ 检索（Retrieval）** 三类操作，落到 Markdown 文件操作上：
- **形成**：会话结束/任务节点时，对原始交互做语义摘要、知识蒸馏（提炼 insight）、结构化（写入约定的 MD 结构），而非全量日志堆积。
- **演化**：定期 **巩固**（合并相似条目、提炼更高层经验）、**更新**（检测并消解新旧规则冲突——直接服务「纠错闭环」）、**遗忘**（时间/频率/重要性三轴剪枝，但保留长尾关键知识，避免误删）。
- **检索**：分层路由 + 查询改写 + 重排过滤，按角色/任务注入精简上下文。
> 关键提醒：短期/中期/长期不必是死板的物理模块，而应由这三算子的调用时机（任务内 vs 跨任务、即时 vs 任务边界 vs 周期性巩固）自然涌现——这正好支撑「三层记忆」的弹性设计。

### 2. 三层记忆架构的功能映射
将「短期/中期/长期」对齐到综述的功能分类，使每层目标清晰：
- **短期 = 工作记忆（Working Memory）**：单任务草稿区，重点做输入压缩、观察抽象、层级折叠（Context-folding/AgentFold 思路），防止超出 CLI Agent 上下文窗口。
- **中期 = 用户事实记忆 + 案例型经验**：跨会话但仍较具体的用户习惯、最近轨迹；保证一致性/连贯性/适应性三特性，直接对应「用户习惯学习」。
- **长期 = 策略型 + 技能型经验 + 环境事实记忆**：沉淀可迁移的 Workflows/Patterns/Insights 与可执行 Skills，对应「技能自动迭代」。

### 3. 经验记忆三层抽象直接指导「技能自动迭代」
Case → Strategy → Skill 的抽象阶梯是技能进化的理论框架：
- 先存**案例**（成功/失败原始轨迹）→ 蒸馏出**策略**（Insights/Workflows/Patterns，可编辑可审计的 MD 规则）→ 固化为**技能**（可调用、结果可验证、可组合的脚本/命令/MCP 工具）。
- 参考 Voyager（增长式技能库）、AWM（工作流归纳）、ReasoningBank（成败皆抽象）、Darwin Gödel Machine（经验证后自改写）的演进路径。
- 三准则用于把控技能质量：**可被 Agent 调用、结果可验证、可与其他技能组合**。

### 4. 多 Agent 共享记忆设计要点（角色分工框架的关键）
- 避免「全局裸共享」带来的记忆杂乱与写竞争；采用 **Agent 感知共享记忆**：读写按角色、专长、信任度条件化（权限/角色感知的访问控制）。
- 借鉴 G-Memory 思路：维护**多粒度图**（如交互层/查询层/洞察层），为不同角色 Agent 提供定制粒度的记忆视图。
- 关注**跨 Agent token 预算分配**（参考 RCR-Router）与角色交接（role handoff）时的共享上下文一致性。

### 5. Markdown 驱动恰好契合「可信记忆」与「自动化管理」前沿
- Token 级记忆的**透明、可编辑、可解释**特性是 MD 文件的天然优势——便于审计、调试、用户共管，直接对接综述的「OS 式可信记忆：分段、版本控制、可审计、Agent 与用户共管」愿景（可用 git 做版本控制）。
- 把「增删改查」做成 Agent 的**显式工具调用**（自动化记忆管理前沿），让 Agent 自己推理记忆操作，比外部硬编码规则更连贯透明——这与 Markdown 文件操作天然吻合。

### 6. 纠错闭环 = 记忆更新（Updating）+ 重要性遗忘 + 反思巩固
- 纠错本质是 **Updating** 的冲突消解：用户纠正 → 检测与既有记忆冲突 → 修正/覆盖。
- 配合**反思式巩固**（Reflexion/ExpeL 思路）把失败经验抽象为 insight 沉淀长期层。
- 可探索类脑**离线巩固**：在空闲/任务边界做周期性记忆重组与剪枝，而非纯在线更新。

### 7. 评估可复用现成 benchmark
开发时可借 LongMemEval（长记忆）、SWE-bench Verified（代码 Agent）、MemoryAgentBench（自进化）验证记忆有效性；开源框架 Mem0/MemOS/Zep/MIRIX 可作为底层存储与检索的参考实现或直接集成对象。
