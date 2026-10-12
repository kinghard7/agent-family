论文概要：
标题：Agent Drift: Quantifying Behavioral Degradation in Multi-Agent LLM Systems Over Extended Interactions
作者：Abhishek Rath
arXiv：2601.04170 (v1, 2026 年 1 月 7 日)
链接： https://arxiv.org/pdf/2601.04170
核心贡献与摘要
这篇论文首次系统性地提出了Agent Drift（代理漂移）概念，定义为：在长时间交互序列中，多 Agent LLM 系统出现的行为逐步退化现象，包括决策质量下降和 Agent 间一致性变差。
论文指出，虽然多 Agent 系统在短期复杂任务上表现出色，但长期稳定性（尤其是跨数月甚至更长的实际部署）被严重忽视。作者建立了理论框架，并通过仿真实验量化了这一问题。
三种主要漂移表现形式：

Semantic Drift（语义漂移）：输出逐渐偏离原始意图，但仍语法正确。
Coordination Drift（协调漂移）：多 Agent 共识机制失效，导致冲突、冗余或协作效率低下。
Behavioral Drift（行为漂移）：出现初始设计中不存在的意外策略或行为模式。

主要内容与创新点
1. Agent Stability Index (ASI)
论文最核心的贡献是提出 Agent Stability Index (ASI) —— 一个复合指标，用于量化漂移。

覆盖 12 个维度，分为四大类：
Response Consistency（响应一致性）
Tool Usage Patterns（工具使用模式）
Inter-Agent Coordination（Agent 间协调）
Behavioral Boundaries（行为边界）

ASI 值范围 [0,1]，低于 0.75 且持续下降即视为显著漂移。
通过滚动窗口计算，可在生产环境中实时监控。

2. 实验与发现

使用 LangGraph 等框架模拟了企业自动化、金融分析、合规监控等场景，共数千个工作流，交互次数最多达 1800+ 次（模拟 3–18 个月）。
结果显示：未控制的 Agent Drift 会导致任务完成准确率显著下降，并大幅增加人工干预需求。
漂移是渐进且隐蔽的，短期评估难以发现。

3. 缓解策略（Mitigation Strategies）
作者提出并理论验证了三种方法：

Episodic Memory Consolidation (EMC)：周期性压缩和提炼历史交互，保留精华、去除冗余。
Drift-Aware Routing (DAR)：路由决策时考虑 Agent 的稳定性分数，对漂移严重的 Agent 进行重置。
Adaptive Behavioral Anchoring (ABA)：动态使用早期基准行为的 few-shot 示例进行“锚定”。

这些策略能在显著降低漂移错误的同时，保持系统吞吐量。
对你的 CodeTeam-MD 框架的参考价值
这篇论文与你的长期项目（服务周期 1 年+）高度相关：

你的 Administrator 角色 可以直接参考 ASI 思想，增加漂移监控和定期重置/锚定机制。
correction_cases.md 和 user_preferences.md 可以扩展为漂移检测的一部分。
记忆管理（尤其是 episodic consolidation）与你已设计的长期记忆系统高度契合。
建议在 Orchestrator 中加入 ASI-like 简单指标监控，Administrator 定期生成漂移报告。

这是一篇非常务实的 2026 年早期论文，侧重工程可靠性而非新架构，强烈推荐阅读 PDF（全文约 14 页，阅读负担小）。