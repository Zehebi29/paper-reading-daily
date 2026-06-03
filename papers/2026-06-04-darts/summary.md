# DARTS: Distribution-Aware Active Rollout Trajectory Shaping for Accelerating LLM Reinforcement Learning

**arXiv**: 2605.30859 | **GitHub**: https://github.com/PKU-DAIR/DARTS | **机构**: Peking University (DAIR Lab)

---

## 一句话总结

DARTS 通过主动塑造 LLM RL 训练中 rollout 轨迹的长度分布（而非被动调度长尾），实现了最高 1.77 倍的端到端加速，且不损害模型性能，已被 ICML 2026 接收。

## 问题背景

RL 训练 LLM（如 GRPO/RLVR）时，rollout 阶段的长尾响应长度分布是主要效率瓶颈，消耗超过 70% 的训练时间。现有方案（如 Tail Batching、Partial Rollout）只在调度层面处理长尾——把长序列凑到一起减少 padding 浪费——但没有触及分布本身。更关键的是，长尾不仅存在于不同 prompt 之间（inter-prompt），还存在于单个 prompt 内部（intra-prompt），而 intra-prompt 长尾往往是无效冗余（verbose），不是真正需要的推理深度。

## 核心方法

- **分布感知轨迹采样**：为每个 prompt 生成 M' 条冗余轨迹（M' >= 目标组大小 M），然后用双端采样策略选出最短的 K 条 + 最长的 L 条组成训练组（K >> L），从根源上把分布往简洁方向推
- **自适应冗余分配**：根据每个 prompt 历史响应长度的方差，动态分配冗余轨迹数量——方差大的 prompt（长尾严重）获得更多冗余采样预算，方差小的少分配
- **梯度动力学分析**：对于冗余 prompt，短轨迹通常有高 reward，抬高 baseline 从而抑制 verbose 长尾；对于真正复杂的 prompt，短轨迹 reward 低，降低 baseline 从而鼓励长轨迹
- **系统级优化**：方差引导的尾部剪枝（对长尾严重的 prompt 直接只采最短）+ token 级流式处理（在生成后缀 token 的同时训练前缀 token，重叠 rollout 和 training）

## 关键结果

| 配置 | 加速比 |
|------|--------|
| VeRL baseline | 1.00x |
| + Token-Level Streaming | 1.09x |
| + Distribution-Aware Sampling | 1.40x |
| + Adaptive Allocation | 1.63x |
| Full DARTS (32 GPUs, Qwen2.5-14B) | 1.77x |

- 在 Qwen 系列 3B 到 32B（含 MoE）模型上测试，MATH500、GSM8K、AIME2024/2025、Olympiad 等数学 benchmark 上性能无下降
- 在 BIG-Bench Hard（逻辑推理、规划）和多模态（Geo3K）、代码（Eurus-2-RL-Data）任务上也有效
- 16 GPUs 时 1.21x，64 GPUs 时 1.55x，GPU 越多加速越明显
- 最优采样比 L:K = 1:7

## 亮点

- 把问题从“调度长尾”提升到“消除长尾”，思路更根本
- 双端采样（最短 + 最长）既压缩冗余又保留推理深度，比简单截断优雅
- 基于 VeRL + vLLM，工程落地友好，代码已开源（9 commits）
- ICML 2026 接收，PKU DAIR Lab 出品，质量有保障

## 局限

- 对输出长度固定的 task 或极端稀疏 reward 的场景效果有限
- 需要生成冗余轨迹（M' >= M），增加了单步计算量，但通过整体加速抵消
- 目前主要在数学推理任务上验证，其他 domain 的泛化需要更多实验

## 个人评价

这篇论文的核心洞察很有价值：与其在调度层面和长尾做斗争，不如直接改变长尾的分布。双端采样策略特别巧妙——选最短和最长而非中间值，从梯度角度自然地让模型学会简洁。对于做 LLM RL 训练效率优化的研究者来说，这是一个即插即用的加速方案。PKU DAIR Lab 的工程能力也值得信赖，代码基于 VeRL 框架，改起来应该不难。

不过要注意，1.77x 的加速是在 VeRL baseline 上测的，如果已经在用 Tail Batching 等优化，实际增益可能是 1.43x（论文也给了这个对比）。另外，这个方法本质上是在牺牲一些探索多样性来换取效率——对于需要大量探索的 task（如全新 domain），冗余采样可能反而有帮助，但双端压缩可能会损失一些有价值的中间长度轨迹。
