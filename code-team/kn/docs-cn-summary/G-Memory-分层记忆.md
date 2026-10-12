# G-Memory：面向多 Agent 系统的分层记忆

## 元信息

- **英文原标题**：G-Memory: Tracing Hierarchical Memory for Multi-Agent Systems
- **arXiv 链接**：https://arxiv.org/abs/2506.07398
- **代码仓库**：https://github.com/bingreeky/GMemory
- **作者机构**：NUS、同济大学、UCLA、A*STAR、NTU（Guibin Zhang、Muxin Fu 等）
- **一句话定位**：一套受「组织记忆理论」启发、即插即用（plug-and-play）的多 Agent 系统（Multi-Agent System, MAS）分层图记忆框架，用 **洞见图 / 查询图 / 交互图** 三层图结构来存储、检索并演化多 Agent 协作轨迹，让 Agent 团队具备跨任务自我进化能力。

---

## 概要

现有 LLM 驱动的多 Agent 系统（MAS）虽在认知与执行能力上远超单体 Agent，但其**自我进化（self-evolving）能力受制于过于简陋的记忆架构**：要么完全忽略 Agent 之间细腻的协作轨迹，要么只把最终结果（artifact）压缩存储，缺乏跨任务（cross-trial）与角色定制化的记忆。G-Memory 借鉴组织记忆理论，提出一个**三层图层级（three-tier graph hierarchy）**：洞见图（insight graph）、查询图（query graph）、交互图（interaction graph）。新任务到来时，它先在查询图上做粗粒度检索，再做**双向遍历（bi-directional traversal）**——向上抽取可泛化的高层洞见，向下提炼精炼的协作轨迹，为每个 Agent 按角色定制记忆；任务完成后三层图协同更新，沉淀团队知识。在 5 个 benchmark、3 种 LLM、3 种 MAS 框架上，无需改动原框架即可将具身任务成功率最高提升 20.89%、知识问答准确率最高提升 10.12%，且 token 开销可控。

---

## 关键信息

### 1. 三层图结构（Three-tier Graph Hierarchy）

记忆按抽象层次自上而下分为三层，对应组织记忆理论中「先用宽泛模式定位、再细粒度访问」的检索逻辑。

| 层级 | 名称 | 定义 | 作用 |
|------|------|------|------|
| 顶层 | **洞见图（Insight Graph）** `G_insight = (I, E_i)` | 节点 `ι_k = (κ_k, Ω_k)`：洞见内容 `κ_k` + 支撑它的查询集合 `Ω_k`；超边 `(ι_m, ι_n, q_j)` 表示洞见 `ι_m` 通过查询 `q_j` 为 `ι_n` 提供语境 | 抽象、可泛化的高层经验/教训（如「下结论前先核查所有可能位置」），提供跨任务知识，指导 MAS 的策略、分工、任务分解 |
| 中层 | **查询图（Query Graph）** `G_query = (Q, E_q)` | 节点 `q_i = (Q_i, Ψ_i, G_inter)`：原始查询 + 任务状态 `Ψ ∈ {Failed, Resolved}` + 关联交互图；边 `E_q` 编码查询间语义关系 | 存储历史任务及元信息，作为检索入口；其拓扑结构支持超越「纯 embedding 相似度」的检索 |
| 底层 | **交互图 / 话语图（Interaction / Utterance Graph）** `G_inter = (U, E_u)` | 节点 `u_i = (A_i, m_i)`：发言 Agent + 文本内容；边按时间/启发关系连接（`u_j` 传递并启发 `u_k`） | 保存 Agent 间细粒度的原始通信日志，记录导致成功或失败协作的具体推理模式 |

### 2. 记忆组织与检索机制（核心工作流，三步）

**① 粗粒度检索（Coarse-grained Retrieval）**
- 用 embedding（实现为 `all-MiniLM-L6-v2`）在查询图上做相似度检索，取 top-k 历史查询 `Q_S`（k 取 1~2 最优）。
- 再做**跳数扩展（hop expansion）**：把 `Q_S` 的 1-hop 邻居并入，得到 `Q̃_S`。实验显示 **1-hop 效果最好**，2/3-hop 反而引入噪声、损害性能。

**② 双向记忆遍历（Bi-directional Memory Traversal）** —— 最关键的设计
- **向上遍历（query → insight）**：通过「查询→洞见投影器」找出支撑查询集与 `Q̃_S` 有交集的所有洞见节点 `I_S`，获取高层指导。
- **向下遍历（query → interaction）**：用 LLM 评分器对历史查询排序取 top-M（M 取 2~5），再用 **LLM 图稀疏器（graph sparsifier）** 把完整交互图压缩成「核心子图」`Ĝ_inter`，只保留关键对话元素，避免 long-context 信息过载。
- **角色定制注入**：算子 `Φ` 评估每条洞见与稀疏交互图相对于「该 Agent 角色 `Role_i` + 当前任务 Q」的效用，为每个 Agent 初始化**专属记忆 `Mem_i`**（角色特定记忆线索 + 交互片段 + 摘要）。这是 MAS 区别于单 Agent 记忆的本质点。

**③ 分层记忆更新（Hierarchy Memory Update）** —— 任务执行后，依环境反馈（执行状态、token 用量等）协同更新三层：
- **交互层**：追踪各 Agent 话语，重建交互图并存储。
- **查询层**：新建查询节点 `q_new`，与 top-M 相关历史查询、以及支撑本次所用洞见的查询集合建立边。
- **洞见层**：用摘要函数 `J` 从本次轨迹蒸馏出新洞见 `ι_new`，与本次用到的旧洞见建立超边；同时把 `q_new` 加入旧洞见的支撑查询集 `Ω_k`，反映其相关性。
- 这一持续更新循环让 G-Memory 跨 episode 不断学习与精炼集体记忆，实现「群体知识的制度化（institutionalization）」。

### 3. 与多 Agent 协作的结合方式

- **即插即用**：作为插件，在 MAS 遇到新查询时触发，无需改动原框架；调用时机可灵活配置（任务开始/每轮对话/特定 Agent）。
- **角色感知**：MAS 中每个 Agent 由四元组 `C_i = (Base_i, Role_i, Mem_i, Plugin_i)` 描述，G-Memory 按 `Role_i` 提供差异化记忆，而非给所有 Agent 灌同样的上下文。
- **提供可执行指导**：检索结果转化为分工建议、任务分解、过往失败教训等，直接喂给团队。
- **多 Agent 轨迹比单 Agent 长得多**（论文指出 token 量可达单 Agent 的 10 倍），故必须做抽象+稀疏化，而非朴素地把全部长上下文塞回去。

### 4. 关键技术点

- 三层图层级 + 超边连接（洞见间通过查询互相语境化）。
- 双向遍历：上抽象、下细节，多粒度记忆支持。
- LLM 图稀疏器做轨迹压缩（trajectory condensation），既保细节又控 token。
- 基于环境反馈（成功/失败状态）的闭环更新，失败也沉淀为「失败教训」。
- 角色定制的记忆注入算子 `Φ`。

### 5. 实验与评估结果

**实验配置**
- **5 个 benchmark / 3 个领域**：知识推理（HotpotQA、FEVER）、具身动作（ALFWorld、SciWorld）、博弈（PDDL）。
- **3 个 MAS 框架**：AutoGen、DyLAN、MacNet。
- **3 种 LLM backbone**：Qwen-2.5-7b、Qwen-2.5-14b、gpt-4o-mini。
- **基线**：单 Agent 记忆（No-memory、Voyager、MemoryBank、Generative Agents）+ 多 Agent 记忆（MetaGPT-M、ChatDev-M、MacNet-M）。

**主要结论（Takeaways）**
- **➊ 全面提升**：在所有任务域和 MAS 框架上一致提升。最高亮点——用 Qwen-2.5-14b 时，G-Memory 把 MacNet 在 ALFWorld 上的成绩从 **58.21% 提升到 79.10%（+20.89%）**；gpt-4o-mini 下平均提升 +8.91%（AutoGen，48.27→57.18）。知识问答最高 +10.12%。
- **➋ MAS 需要专门记忆**：多数现有记忆机制在 MAS 上不稳定甚至**反向掉点**（如 Voyager/MemoryBank 在 PDDL 上分别让 AutoGen 掉 4.17%/1.34%；ChatDev-M 在 MacNet+SciWorld 掉 2.32%）。原因是它们缺乏角色特定支持、记忆范围太窄（只存执行结果）。这反证了 G-Memory 三大特性的必要性：**角色特定记忆线索、抽象高层洞见、轨迹压缩**。
- **➌ 高性价比**：在 PDDL+AutoGen 上比 no-memory 提升 10.32%，仅增加 1.4×10⁶ token；而 MetaGPT-M 多花 2.2×10⁶ token 才提升 4.07%。
- **消融实验**：洞见层与交互层都有贡献，去掉任一层都掉分；**交互层（细粒度对话轨迹）影响略大于洞见层**（去掉交互层 AutoGen 掉 4.47%，去掉洞见层掉 3.95%）。
- **参数敏感性**：hop 取 1、k 取 1~2 最优，过大会引入无关噪声。

---

## 总结

**核心贡献**
1. **瓶颈识别**：系统性指出现有 MAS 自我进化能力的根本瓶颈在于过度简化的记忆架构（忽略协作轨迹、缺乏跨任务与角色定制）。
2. **方案**：提出 G-Memory——洞见/查询/交互三层图层级 + 双向遍历检索 + 分层协同更新，为每个 Agent 提供从抽象洞见到细粒度协作片段的定制化、分层记忆，并跨 episode 动态演化。
3. **实证**：跨 5 benchmark、3 LLM、3 MAS 框架验证，即插即用、显著提升（具身任务最高 +20.89%），且 token 友好。

**局限**
- 仅在 3 个领域、5 个 benchmark 上验证，缺乏更多样任务（如医疗问答）的检验。
- 重度依赖 LLM 做评分、稀疏化、摘要，这些子调用的成本与可靠性论文未深入分析。
- 双向遍历与三层更新逻辑较复杂，工程落地成本与超参（hop、k、M）调优有一定门槛。

---

## 对 CodeTeam-MD 的启示

G-Memory 的「分层图记忆」与 CodeTeam-MD 的「分层团队 + 三层长期记忆」在结构上高度同构，可直接借鉴：

### 1. 三层图 → 三层记忆 + 分层团队的天然对应
- **洞见图 ↔ 项目级/团队级长期记忆**：沉淀可泛化的工程洞见与失败教训（如「改 API 前先核查调用方」「Tester 报错前先确认环境」），由 Architect 层消费，对应 CodeTeam-MD「用户习惯学习与纠错闭环」中的经验沉淀。
- **查询图 ↔ 任务/需求索引层**：把历史需求（PM 输入的 query）按语义连成图，新需求来时先做相似检索 + 1-hop 扩展，复用类似需求的设计与实现路径。
- **交互图 ↔ 角色协作轨迹（Markdown 驱动）**：CodeTeam-MD 各角色之间通过 Markdown 文档交接，这些交接记录正好可建模为「话语图」节点（发言角色 + 内容 + 时序/启发边），存为可检索的协作轨迹。

### 2. 双向遍历适配「Architect → Coder → Tester」分层
- **向上遍历**为 Architect 提供高层洞见（架构决策、踩坑教训）；**向下遍历**为 FullstackDev/Tester 提供具体的历史实现/测试轨迹片段。两层粒度天然匹配「先决策、后执行」的分层流水线。

### 3. 角色定制记忆注入（最值得借鉴的一点）
- 论文证明「给所有 Agent 灌同样上下文」会掉点，必须**按 Role 定制记忆**。CodeTeam-MD 的 PM / Architect / FullstackDev / Tester / Administrator 各有职责，应为每个角色注入与其职责相关的记忆切片（PM 看需求历史、Architect 看架构洞见、Tester 看历史缺陷模式），而非共享同一份上下文。

### 4. 轨迹压缩控制 token 成本
- CLI Coding Agent（Codex/Claude Code）多轮协作轨迹极长（论文指出可达单 Agent 的 10×）。借鉴「LLM 图稀疏器」思路，对 Markdown 协作日志做核心子图提取/压缩后再复用，而非全量回灌，可显著省 token。

### 5. 基于反馈的闭环更新 = 纠错闭环的实现蓝本
- G-Memory 任务后依环境反馈（成功/失败、token、工具调用）更新三层图，把失败沉淀为「失败教训」。这正是 CodeTeam-MD「纠错闭环」可落地的机制：每次构建/测试结果反馈后，更新对应层的记忆，尤其把 Tester 发现的问题蒸馏为 Architect 层洞见。

### 6. 即插即用 + 超参经验
- G-Memory 不改原 MAS 框架即可叠加，提示 CodeTeam-MD 的记忆层应设计为与角色编排解耦的独立模块。
- 可直接复用其超参经验：**hop=1、k∈{1,2}**，避免检索过宽引入无关噪声。
