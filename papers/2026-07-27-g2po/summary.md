# G2PO: Group-Graph Policy Optimization for Long-Horizon Agentic Reinforcement Learning

**arXiv**: 2606.22995 | **GitHub**: https://github.com/Nala-YN/G2PO | **机构**: Peking University + Microsoft Research

---

## 一句话总结

G2PO 将传统线性交互轨迹转换为全局状态转换图，通过图结构中的边级优势估计来解决长治期 agentic RL 中的奖励稀疏和信用分配问题，在多个基准上达到最大 22.2% 的成功率提升相比 GRPO。

## 问题背景

LLM agent 在长治期任务中需要与环境进行大量交互回合，但现有的 agentic RL 方法存在两个核心瓶颈：奖励稀疏和延迟（反馈常延迟数十步）、以及信用分配粗糙——现有步级方法仍将探索视为孤立线性轨迹，忽略了状态转换之间的图结构关系，导致状态价值估计方差高、信用分配短视。

## 核心方法

1. **图转换**：将多条并行线性交互轨迹显式转换为全局状态转换图（state-transition graph），相同观察被聚合为同一节点。

2. **组聚合状态价值估计**：通过聚合跨越不同轨迹的相同观察，引入组聚合状态价值估计，降低采样方差和轨迹依赖偏差。

3. **边级优势估计**：将 agent 的动作重新定义为状态节点之间的边，并基于全局图结构提出边中心的优势估计策略。

4. **全局 TD 错误标准化**：通过在整个图上对 Temporal Difference 错误进行全局标准化，明确识别和优先处理驱动绝对任务进展的关键转变。

## 关键结果

- 论文报告 G2PO 在 WebShop、ALFWorld、AppWorld 三个长治期 agent 基准上显著优于 SOTA prompt-based 和 RL 基线，实现相比 GRPO 最高 22.2% 的成功率提升。
- 具体基准细分结果见论文图表，所有实验均采用相同访调和训练设置进行公平对比。

## 亮点

1. 独创性强：首次将图结构引入 agentic RL 的信用分配，理论幅度清晰，工程实现完整。
2. 实验充分：在 WebShop/ALFWorld/AppWorld 三个标准长治期 agent 基准上进行评估，结果一致优于 GRPO 和其他先进方法。
3. 源码开放：GitHub Apache-2.0 开源，且来自 Microsoft Research （Furu Wei, Qi Zhang 等），可信度高。
4. 实用性强：critic-free、低内存消耗，与 GRPO 相比添加计算开销极小，容易集成到现有管道。

## 局限

1. Github 仅 2 stars（论文新发布），社区验证不足，是否在更大规模的模型上稳定还待验证。
2. 缺乏更复杂的现实场景测试（如 SWE-bench, Terminal-Bench 等），仅在传统体现任务上验证。
3. 拓扑图构建的计算开销随状态空间增大而增加，在极端大状态空间场景下可能成为瓶颈。

## 个人评价

这篇论文给了我一个很大的启发：当所有人都在做步级 GRPO/更粗糙的信用分配时，G2PO 转变思路，从图论角度重新定义信用分配问题。不同轨迹中出现相同观察状态是常见的，但之前的方法都没有利用这个结构信息。来自 Microsoft Research 的 Furu Wei 和 Qi Zhang 的团队出品，质量有保障。

对我们的 GWS 项目来说，这种图结构信用分配的思想可能有借鉴意义——焊缝路径规划也是一个多步骤决策过程，如果能用图结构的思想来做步骤级的信用分配，可能会有意想不到的效果。
