# Is One Layer Enough? Training A Single Transformer Layer Can Match Full-Parameter RL Training

**arXiv**: 2607.01232 | **GitHub**: 无 | **机构**: University of Minnesota + Peking University + Amazon

---

## 一句话总结

这篇论文发现 RL post-training 的收益高度集中在 transformer 的少数中间层，单层训练即可恢复甚至超越全参数 RL 训练的 gains，并据此提出 layer-aware 训练策略。

## 问题背景

RL post-training（如 GRPO）已成为 LLM 对齐的核心方法，但对“RL 的改进究竟分布在网络的哪些部分”缺乏理解。现有方法默认所有层均匀贡献，但这种假设从未被严格检验过。

## 核心方法

1. 提出 **layer contribution** 指标：C(k) = (单训练第 k 层的收益) / (全参数训练的收益)，量化每层对 RL gains 的贡献占比。
2. 在 7 个模型（Qwen3-1.7B/4B/8B, Qwen2.5-Math-1.5B, Qwen2.5-1.5B/3B-Inst, DeepSeek-Distilled-Qwen-7B）、3 种 RL 算法（GRPO, GiGPO, Dr. GRPO）、3 个任务域（数学推理、代码生成、agentic decision-making）上进行系统性消融实验。
3. 基于分析结果设计 layer-aware 训练策略：只训练高贡献层或对高贡献层增加学习率。

## 关键结果

| 实验设置 | 结果 |
|---------|------|
| 单层 vs 全参数 RL | 单层训练恢复大部分收益，某些情况 C(k) > 1（超越全参数） |
| 高贡献层位置 | 稳定位于 transformer 中间层（“Goldilocks Zone”） |
| 层排名跨任务/跨模型/跨算法 | Spearman 相关性高，模式一致 |
| Layer-aware 训练策略 | 一致优于标准全参数 RL |
| Layer-specialized 模型集成 | 通过互补行为带来额外增益 |

## 亮点

- **颠覆性发现**：RL 收益并非均匀分布在所有层，而是高度集中——单层即可 match 全参数训练
- **跨设置稳定**：7 个模型 x 3 种算法 x 3 个任务域，模式高度一致，结果可信度高
- **实用价值高**：layer-aware 训练比全参数训练效果更好且更省资源
- **理论启发**：质疑了“全参数更新必要”的隐含假设，为理解 RL 如何修改 LLM 提供新视角

## 局限

- 只在 Qwen 系列和 DeepSeek-distilled Qwen 上验证，未覆盖更大模型（如 70B+）或其他架构（如 MoE）
- layer contribution 指标依赖逐层独立训练，未考虑层间交互的非线性效应
- agentic decision-making 任务的具体 benchmark 未详细披露，实验范围有限
- 无公开代码和实验配置，复现门槛较高

## 个人评价

这篇论文的发现非常漂亮——“RL gains are concentrated in a single layer”这个结论如果成立，对 RL post-training 的效率有巨大的实际意义。Layer-aware training 比全参数训练效果更好，意味着我们一直在浪费大量计算资源训练那些贡献很小的层。此外，这个发现也和 LoRA 等参数高效微调方法的成功相呼应——优化的有效参数空间可能本来就很小。不过论文只在中小规模模型上验证，扩展到 70B+ 或 MoE 架构后规律是否仍然成立需要进一步验证。如果规律普遍成立，这可能会催生新一代的 layer-aware RL 训练 framework。
