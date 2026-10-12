# CodeTeam-MD 论文导读索引

本目录收录 7 篇与 **CodeTeam-MD** 多 Agent 协同框架强相关的论文中文概要。每篇统一结构：**概要 → 关键信息 → 总结 → 对 CodeTeam-MD 的启示**（非逐字翻译，术语保留英文原文）。

> CodeTeam-MD 核心特点：Markdown 驱动、角色分工（PM / Architect / FullstackDev / Tester / Administrator）、三层长期记忆、用户习惯学习与纠错闭环、技能自动迭代，运行在 CLI Coding Agent（Codex / Claude Code）之上。

---

## 一、按主题分组

### 综述（打地基，先读）
| 文档 | 论文 | 一句话定位 |
|---|---|---|
| [代码生成Agent综述](代码生成Agent综述.md) | A Survey on Code Generation with LLM-based Agents（[2508.00083](https://arxiv.org/abs/2508.00083)） | 单/多 Agent 代码生成的完整 taxonomy 与全 SDLC 应用 |
| [多Agent协作机制综述](多Agent协作机制综述.md) | Multi-Agent Collaboration Mechanisms（[2501.06322](https://arxiv.org/abs/2501.06322)） | 按结构/策略/协调五维度刻画 MAS 协作通道 |
| [AI-Agent记忆综述](AI-Agent记忆综述.md) | Memory in the Age of AI Agents（[2512.13564](https://arxiv.org/abs/2512.13564)） | Forms–Functions–Dynamics 三维统一记忆框架 |

### 记忆系统（落地三层记忆）
| 文档 | 论文 | 一句话定位 |
|---|---|---|
| [MIRIX-多Agent记忆系统](MIRIX-多Agent记忆系统.md) | MIRIX（[2507.07957](https://arxiv.org/abs/2507.07957)） | 六类记忆 + 八 Agent，Active Retrieval 路由检索 |
| [G-Memory-分层记忆](G-Memory-分层记忆.md) | G-Memory（[2506.07398](https://arxiv.org/abs/2506.07398)） | 三层图记忆（洞见/查询/交互）+ 双向遍历检索 |

### 工程框架（落地角色分工）
| 文档 | 论文 | 一句话定位 |
|---|---|---|
| [ALMAS-多Agent软件工程框架](ALMAS-多Agent软件工程框架.md) | ALMAS（[2510.03463](https://arxiv.org/abs/2510.03463)） | 敏捷团队式六角色 + 两阶段工作流 |

### 长期可靠性（落地漂移监控）
| 文档 | 论文 | 一句话定位 |
|---|---|---|
| [Agent Drift 代理漂移](Agent%20Drift%20代理漂移.md) | Agent Drift（[2601.04170](https://arxiv.org/pdf/2601.04170)） | 量化长期交互行为退化，提出 ASI 指标 + 三种缓解策略（EMC/DAR/ABA） |

---

## 二、按 CodeTeam-MD 设计决策查阅

| 你在设计…… | 主要参考 | 核心借鉴点 |
|---|---|---|
| **角色分工 / 团队模板** | ALMAS、协作机制综述 | ALMAS 六角色（Sprint/Supervisor/Summary/Control/Code/Peer）直接对应 PM/Architect/Dev/Tester；综述提醒「协作通道质量 > Agent 数量」 |
| **三层记忆架构** | 记忆综述、MIRIX、G-Memory | 短期/中期/长期是「调用模式」的涌现；MIRIX 六类记忆细化分层；G-Memory 三层图匹配 Architect→Coder→Tester |
| **技能自动迭代** | MIRIX、代码生成综述 | MIRIX 的 Procedural Memory（workflow/guide/script）即技能载体；综述的「自演化」工作流支撑技能进化 |
| **Administrator 定位** | MIRIX、协作机制综述 | MIRIX 的 Meta Memory Manager = 路由协调中枢；综述建议 Administrator 定位为 Orchestrator |
| **纠错闭环** | MIRIX、G-Memory、记忆综述 | MIRIX「用户纠错→精确更新对应记忆」；G-Memory 反馈闭环更新；记忆综述「纠错=记忆 evolution 算子」 |
| **记忆检索 / 上下文注入** | MIRIX、G-Memory | Active Retrieval 先定话题再检索；粗检索→精检索两段式；按角色定制记忆注入，控 token 预算 |
| **Markdown 即协作介质** | 代码生成综述、协作机制综述 | 综述的「黑板模型」≈ 共享 Markdown；显式声明协作通道 |
| **长期漂移监控 / 定期重置** | Agent Drift、MIRIX | ASI 复合指标（12 维 4 类，<0.75 持续下降即漂移）；Administrator 定期生成漂移报告并对漂移 Agent 重置/锚定；EMC 周期性压缩历史交互对应 episodic consolidation |

---

## 三、推荐阅读顺序

1. **代码生成Agent综述** —— 建立编码 Agent 全局认知（单/多 Agent、规划/工具/记忆/反思）。
2. **多Agent协作机制综述** —— 理解 orchestrator + worker 的协调与通信设计。
3. **AI-Agent记忆综述** —— 掌握记忆的统一框架与生命周期算子。
4. **MIRIX + G-Memory** —— 两套可直接借鉴的记忆落地方案。
5. **ALMAS** —— 现成的敏捷团队角色模板，对照 CodeTeam-MD 角色定义。
6. **Agent Drift** —— 长期部署（1 年+）的可靠性保障，ASI 监控 + 漂移缓解。

---

## 附：待补充

- **AgentSpawn: Adaptive Multi-Agent Collaboration Through Dynamic Spawning**（[2602.07072](https://arxiv.org/abs/2602.07072)）——「动态生成 sub-agents、按需分配上下文」，对应 CLI 实例动态扩展。该论文 PDF 暂未收录在 `docs-en/`，待补充后再生成概要。
