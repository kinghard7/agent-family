# 多 Agent 协同 Agent-Team 开发调研报告（CLI 编码 Agent 导向）

**报告日期**：2026 年 6 月  
**作者**：Grok AI 调研总结  
**目标**：针对**依托现有 CLI 编码 Agent**（如 OpenAI Codex CLI、Anthropic Claude Code、CodeBuddy 等）构建多 Agent 协同系统（Agent-Team）。强调**类似于 Skill 的可复用模块化设计**，支持角色分工、并行工作、记忆共享，并可集成到更大框架中（如自定义 Skill/Tool）。

**核心理念**：  
- 以 CLI 工具（如 Claude Code / Codex CLI）作为基础执行单元，通过 orchestrator 协调多实例（git worktree 并行、sub-agents）。  
- 设计成**Skill-like 模块**：可封装为独立工具、插件或 MCP 服务，便于复用和扩展。  
- 重点解决编码场景：代码生成、审查、重构、测试、调试的团队协作。

**推荐阅读人群**：后端/全栈开发者、AI 工具构建者、希望扩展现有 CLI 编码 Agent 成生产级团队的工程师。

---

## 1. 核心挑战（CLI 编码场景）
- CLI 实例并行管理（git worktree、独立 session）
- 角色分工与通信（orchestrator → sub-agents）
- 记忆共享 vs 隔离（跨 session 持久化）
- Skill-like 封装（工具调用、MCP 集成、可复用）
- 容错与审查（人类-in-the-loop）

---

## 2. 核心综述与框架论文（必读）

### A Survey on Code Generation with LLM-based Agents
- **链接**： [arXiv:2508.00083](https://arxiv.org/abs/2508.00083)  
  **PDF**： [下载](https://arxiv.org/pdf/2508.00083.pdf)
- **概要**：系统梳理单/多 Agent 代码生成技术，涵盖架构、协作、多 Agent 协调复杂度。
- **为什么选择**：直接针对代码生成，提供 CLI Agent 协作的理论基础。
- **适合人群**：所有构建编码 Agent-Team 的开发者。

### Multi-Agent Collaboration Mechanisms: A Survey of LLMs
- **链接**： [arXiv:2501.06322](https://arxiv.org/abs/2501.06322)  
  **PDF**： [下载](https://arxiv.org/pdf/2501.06322.pdf)
- **概要**：按结构、策略、协调协议分类 MAS 协作机制。
- **为什么选择**：指导如何用 CLI 实例构建 orchestrator + worker 模式。
- **适合人群**：架构设计者。

### Memory in the Age of AI Agents: A Survey
- **链接**： [arXiv:2512.13564](https://arxiv.org/abs/2512.13564)  
  **PDF**： [下载](https://arxiv.org/pdf/2512.13564.pdf)
- **概要**：Agent 记忆统一框架，重点多 Agent 扩展。
- **为什么选择**：CLI 多 session 需要持久化记忆共享，此文提供设计蓝图。
- **适合人群**：需要长期上下文的团队。

---

## 3. CLI 多 Agent 编码实践与记忆论文

### MIRIX: Multi-Agent Memory System for LLM-Based Agents
- **链接**： [arXiv:2507.07957](https://arxiv.org/abs/2507.07957)  
  **PDF**： [下载](https://arxiv.org/pdf/2507.07957.pdf)
- **概要**：模块化记忆系统（Core/Episodic/Semantic/Procedural 等），动态协调多 Agent。
- **为什么选择**：完美适配 CLI 多实例的记忆共享，可封装为 Skill。
- **适合人群**：需要跨 CLI session 知识复用的开发者。

### G-Memory: Tracing Hierarchical Memory for Multi-Agent Systems
- **链接**： [arXiv:2506.07398](https://arxiv.org/abs/2506.07398)  
  **PDF**： [下载](https://arxiv.org/pdf/2506.07398.pdf)
- **概要**：三层图结构层次化记忆，适合分层团队（Architect → Coder → Tester）。
- **为什么选择**：层次设计天然匹配 CLI orchestrator + sub-agents。
- **适合人群**：复杂编码任务团队。

### ALMAS: an Autonomous LLM-based Multi-Agent Software Engineering Framework
- **链接**： [arXiv:2510.03463](https://arxiv.org/abs/2510.03463)  
  **PDF**： [下载](https://arxiv.org/pdf/2510.03463.pdf)
- **概要**：敏捷团队式多 Agent 框架（Product Manager、Developer、Tester、Reviewer），适用于真实软件工程。
- **为什么选择**：直接提供 CLI 可实现的角色分工模板。
- **适合人群**：端到端编码工作流构建者。

### AgentSpawn: Adaptive Multi-Agent Collaboration Through Dynamic Spawning
- **链接**： [arXiv:2602.07072](https://arxiv.org/abs/2602.07072)
- **概要**：动态生成 sub-agents，按需分配上下文，支持长时程编码。
- **为什么选择**：解决 CLI 实例动态扩展问题，类似 Skill 按需调用。
- **适合人群**：长任务、多分支开发场景。

---

## 4. 工程实践与框架（Skill-like 集成）

### Towards a Science of Scaling Agent Systems
- **链接**： [arXiv:2512.08296](https://arxiv.org/abs/2512.08296)
- **概要**：大规模 Agent 缩放实验与规律总结。
- **为什么选择**：指导 CLI 实例数量与并行策略（避免性能退化）。
- **适合人群**：生产级部署。

**推荐开源框架（与 CLI 集成）**：
- **CrewAI / AutoGen / LangGraph**：构建 orchestrator，调用 CLI 工具（subprocess 或 MCP）。
- **Claude Code / Codex CLI 原生**：内置 sub-agents、Agent Teams、git worktree 并行。
- **MCP（Model Control Protocol）**：将自定义 Skill 暴露给 Claude Code 等。

**Skill-like 设计建议**：
- 将每个角色封装为独立 CLI Skill（带 manifest、工具列表、记忆 hook）。
- 使用 LangGraph 状态机或 CrewAI Flows 编排。
- 持久化通过文件/MCP/向量 DB 实现跨 session 记忆。

---

## 5. 推荐阅读与开发顺序
1. 综述论文（代码生成调研 + 协作机制）。
2. 记忆与框架（MIRIX、G-Memory、ALMAS）。
3. 实践（AgentSpawn + CLI 框架集成）。
4. 动手：用 CrewAI/LangGraph 包装 Codex/Claude Code CLI 实例，实现一个简单 Skill（e.g. Code-Review-Skill）。

**附加资源**：
- GitHub：YennNing/Awesome-Code-as-Agent-Harness-Papers（编码 Agent 论文列表）
- 实践指南：Claude Code / Codex CLI 官方文档中的 Agent Teams / Multi-Agent 部分
- 社区：Reddit r/ClaudeAI、r/codex 中的并行 CLI 工作流讨论

**趋势**（2026）：CLI-first 多 Agent（worktree + sub-agents）、动态 Skill 封装、MCP 集成成为主流，记忆层次化与动态 spawning 是关键。

---

**报告结束**  
此报告聚焦 CLI 编码场景与 Skill-like 设计，可直接用于开发路线图。如需具体代码示例、Skill 封装模板、或某篇论文详细提取，请告知！