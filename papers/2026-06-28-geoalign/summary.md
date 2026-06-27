# GEOALIGN: Geometric Rollout Curation for Robust LLM Reinforcement Learning

**arXiv**: 2606.26917 | **GitHub**: https://github.com/SYSUzhouting/Trinity-RFT | **机构**: Sun Yat-Sen University + Alibaba Group (ICML 2026)

---

## 一句话总结

GEOALIGN 提出在 LLM 在线 RL 训练中检测和修复“方向不一致”（directional inconsistency）的 rollout，通过轻量级前向投影实现更稳定的策略优化，在数学推理和 RLHF 上都超越 PF-PPO/PAR/PODS/Seed-GRPO 等基线。

## 问题背景

在线 RL 用于 LLM 对齐时，训练不稳定是常见问题。作者发现一个被忽视的失败模式：同一 batch 内，少数高奖励 rollout 在表示空间中产生的偏好更新方向与 batch 多数样本严重冲突，导致高方差和震荡。现有的稳定方法只关注标量 reward 层面的筛选（clipping/shaping/uncertainty weighting），忽略了向量级更新方向的一致性。

## 核心方法

1. **方向不一致检测**：对每个 prompt 的多个 rollout，使用最后一层 hidden states 构造 reward-ordered displacement directions（低到高奖励的隐空间方向向量），在线学习一个投影器来集中这些方向。

2. **Batch Consensus Prototype**：构建 batch 级别的共识原型方向，计算每个 rollout 方向与原型之间的角度偏差，识别方向异常值。

3. **定向修正**：只对检测出的严重方向不一致的 rollout 进行替换（用该 prompt 下方向更一致的替代 rollout），不改变其他 rollout，保持有效 batch size。

4. **前向传播仅一次**：GEOALIGN 只需 forward pass，不涉及梯度计算或额外模型推理，计算开销可忽略。

## 关键结果

| 设定 | 模型 | 指标 | BASE-GRPO | GEOALIGN | 提升 |
|:---|:---|:---:|:---:|:---:|:---:|
| Math (6项基准平均) | Qwen3-1.7B | Avg Acc | 39.81 | **40.44** | +1.6% rel |
| Math (6项基准平均) | Qwen3-4B | Avg Acc | 54.79 | **55.94** | +2.1% rel |
| RLHF (HH-rlhf) | Qwen3-1.7B | Mean Score | 0.8354 | **0.8885** | +6.4% rel |
| RLHF (HH-rlhf) | Qwen3-4B | Mean Score | 0.8672 | **0.8894** | +2.6% rel |

## 亮点

- **问题发现新颖**：首次系统性地识别“方向不一致”这一 LLM RL 训练失败模式，提供了可视化证据（长尾几何分布）
- **方法简洁高效**：forward-pass only 的轻量级插件，不改变训练 pipeline 结构，易于集成到现有框架
- **双域验证**：在数学推理（二进制验证奖励）和 RLHF（连续奖励）两个差异较大的场景中都验证有效
- **ICML 2026 接收**：顶级会议认证 + 代码已开源（SYSUzhouting/Trinity-RFT），可复现

## 局限

- **改进幅度有限**：在已有较强基线上的相对提升在 1.6%-6.4% 之间，不算突破性增益
- **依赖隐空间质量**：方法有效性依赖于模型 hidden states 的表达质量，对较小模型的效果可能更弱
- **实验规模**：只验证了 Qwen3-1.7B 和 Qwen3-4B，更大模型（如 32B/72B）上的效果未知
- **基线和对比范围**：只对比了 GRPO 家族的变体，未与 DPO-based 方法或更近期的先进方法对比

## 个人评价

这篇论文的可贵之处在于它提出了一种“看方向而非看分数”的新视角。传统 RL 训练中大家关注的是 reward 数值的可靠性，而 GEOALIGN 提醒我们：即使 reward 完全正确，不同 rollout 在参数空间中指向的方向可能大相径庭。这个视角对未来 agent 训练特别有意义——agent 任务中的 reward 往往更稀疏、更嘈杂，方向一致性可能是比 reward 本身更可靠的训练信号。插件式设计也让它容易集成到现有 agent RL 框架中。值得关注的是代码仓库名叫 Trinity-RFT，可能后续会有更多相关方法论发布。
