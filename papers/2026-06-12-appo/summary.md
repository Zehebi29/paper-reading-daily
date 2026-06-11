# APPO: Agentic Procedural Policy Optimization

**arXiv**: [2606.12384](https://arxiv.org/abs/2606.12384) | **GitHub**: 无 (work in progress) | **机构**: 未明确标注 (work in progress)

---

## 一句话总结

APPO 将 agentic RL 的分支点选择和信用分配从粗粒度的 tool-call 边界细化到 token 级别的“过程决策点”, 在 13 个 benchmark 上比强基线平均提升约 4 分.

## 问题背景

现有的 agentic RL 方法 (如 GRPO, ARPO) 通常在 tool-call 边界或固定工作流节点上做分支和信用分配. 但论文的 pilot study 发现: (1) 真正影响最终结果的决策点广泛分布在整个生成序列中, 不集中在 tool-call 处; (2) 单纯的 token 熵并不能可靠地标识决策重要性.

## 核心方法

- **Procedure 概念**: 将“过程”定义为序列中潜在的决策点, 即不同续写会导致截然不同下游结果的位置
- **Branching Score (BS)**: 结合 token 熵 (局部不确定性) 和未来感知的策略似然增益, 用 z-score 归一化后相乘, 选出高价值分支位置
- **Procedural Rollout Branching**: 先生成 N 条完整 rollout, 计算 BS 选出 Top-B 个分支点, 从这些点重新采样产生分支
- **Dual-Group Advantage**: 对初始 rollout 和分支 rollout 分别计算 group-relative advantage, 叠加 future-aware advantage
- **理论保证**: Theorem 3.1 证明方差减少; Theorem 3.2 给出策略改进下界

## 关键结果

| Backbone | 基线 (avg) | APPO (avg) | 提升 |
|----------|-----------|-----------|------|
| Qwen2.5-7B-Instruct | ARPO 58.3% | 62.2% | +3.9 |
| Qwen3-8B | 基线 ~55% | ~59% | ~4 |
| Qwen3-14B (GAIA) | ARPO 43.7 | 46.6 | +2.9 |

- 在数学推理 (AIME24/25, MATH500, GSM8K), 知识密集推理 (HotpotQA, 2WikiMultihopQA, MuSiQue, Bamboogle, WebWalker), 深度搜索 (GAIA, HLE, XBench) 三类共 13 个 benchmark 上均有提升
- Pass@K 分析显示 APPO 不仅提升 top-1 准确率, 还改善了有效推理路径的分布
- 训练动态更平滑, 最终奖励更高

## 亮点

- Pilot study 的发现很有洞察力: 决策点不集中在 tool-call, 熵不是好的决策重要性指标
- Branching Score 的设计简洁且有理论支撑
- 实验覆盖面广 (13 benchmark x 4 backbone)
- Dual-Group + Future-Aware Advantage 的组合设计合理, 消融实验显示每个组件都有贡献

## 局限

- 目前是 work in progress (25 页), 没有开源代码
- Branching Score 需要计算每个 token 的未来似然增益, 计算开销可能不小
- 主要在 7B-14B 规模验证, 更大模型上的效果未知
- 与 TRACE (2606.11119) 解决的问题有重叠, 但未做对比

## 个人评价

这篇论文的核心洞察很有价值: 现有 agentic RL 在 tool-call 边界做信用分配是过于粗糙的, 真正重要的决策点散布在整个 thinking span 中. Branching Score 把 token 熵和策略似然增益结合起来做分支点选择, 比单纯用熵或者均匀采样要聪明得多. 理论部分也给方法提供了较好的说服力. 不过作为 work in progress, 代码未开源, 计算开销的讨论缺失, 实际可复现性还需观望.
