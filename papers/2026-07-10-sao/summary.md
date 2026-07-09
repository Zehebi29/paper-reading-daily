# Single-Rollout Asynchronous Optimization for Agentic Reinforcement Learning

**arXiv**: 2607.07508 | **GitHub**: THUDM/slime (SAO 已部署在 slime 框架中) | **机构**: Tsinghua University / THUDM

---

## 一句话总结

SAO 提出单滚动组异步优化替代 GRPO 的组级采样，在 Qwen3-30B-A3B 上 AIME2025 从 80.4% 提升至 97.3%，SWE-Bench Verified 从 23.0% 提升至 29.8%，并已成功部署到 GLM-5.2 (750B-A40B) 的 Agent化 RL 训练管道中。

## 问题背景

当前 LLM 的 RL 管道大多采用同步、批量交替的方式，对于长过程的代理任务效率极低（短得等长的……。同步交替在异步 RL 中不太自然。

## 核心方法

1. **异步单滚动组** - 抛弃 GRPO 的组级多样本采样，改为每个 prompt 只生成一条序列，到达即更新，减少 off-policy 风险

2. **价值模型训练** - 引入实用的价值模型设计（K=2 更快更新频率），用单个补企值自适应滚动组大小

3. **双侧 token 级剪切 (DIS)** - 严格的双侧 token 级别剪切策略 (ε_low=ε_high 对称)，显著提升异步 RL 的训练稳定性

4. **长度自适应 GAE** - 使用 length-adaptive GAE，根据序列长度自动调整 λ 参数

## 关键结果

**Table 1: 数学推理基准 (Accuracy %)**

| Model | AIME2025 | BeyondAIME | HMMT | IMOAnswerBench |
|---|---|---|---|---|
| SFT (w/ python) | 80.4 | 53.3 | 75.2 | 53.3 |
| GRPO (w/ python) | 84.2 | 54.8 | 76.0 | 55.8 |
| **SAO (ours)** | **97.3** | **74.8** | **88.3** | **74.0** |
| GRPO (+ DIS) | 93.5 | 70.8 | 84.0 | 70.0 |

**Table 2: SWE-Bench Verified (Accuracy %)**

| Model | Accuracy |
|---|---|
| Qwen3-30B-A3B | 23.0 |
| + GRPO (w/ DIS) | 27.0 |
| + **SAO (ours)** | **29.8** |

## 亮点

1. 在数学推理上提升显著 - AIME2025 从 80.4% 到 97.3% (+16.9pp)，已超过 Claude-Sonnet-4.5 (87.0) 和 GPT-5 High (94.6)
2. 第一个证明单滚动组可以替代组级采样的工作 - 理论上更简单却更有效
3. 已在生产级模型 GLM-5.2 (750B-A40B) 的训练中验证可行性
4. DIS 剪切策略独立有效 - SAO (w/ DIS only) 在数学上已达 94.2/71.5/86.7/71.3，仍远超 GRPO

## 局限

1. 仅在单一基础模型 (Qwen3-30B-A3B) 上验证，模型规模推广性待研究
2. 价值模型的引入增加了参数量和训练复杂度（相比 GRPO 无价值模型版本）
3. 没有单独的代码仓库，依赖 THUDM/slime 生态

## 个人评价

这篇的角度很精妙 - GRPO 的组级采样确实是为同步批量训练设计的，异步下不自然就是一个“已知但没人说破”的问题。去年 2025 年很多 RL 训练框架还在强行把 GRPO 塞进异步管道，SAO 直接说“别用组了，用单个就好”，并且用价值模型补偿单个样本的方差问题，这个思路很干净。

从结果来看，数学推理 AIME2025 97.3% 这是数据上的飙升，但 SWE-Bench 从 27.0 到 29.8 的提升相对温和（GRPO 从 base 23.0 到 27.0 本身就提升不大），说明编码任务的难度更大，单纯的算法改进幅度有限。对于 GWS 项目，这篇的启示是：如果当前 RL 管道使用的是组级采样且有明显 bottleneck，转单个滚动组 + 价值模型是一个值得尝试的改进方向。
