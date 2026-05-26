# RELEX：只需 15% RLVR 训练步数，就达到完整训练性能

**arXiv**: 2605.21468 | **GitHub**: https://github.com/weizhepei/RELEX | **机构**: UVa & Washington University in St. Louis

---

## 一句话总结

RELEX 发现 RLVR 权重轨迹天然是 rank-1 且线性可预测的。只需观察前 15% 的训练步，就能外推到完整训练性能，在三个 Qwen 模型上匹配甚至超过完整 RLVR 训练的效果。

## 背景

RLVR 已成为提升 LLM 推理能力的主流范式（如 DeepSeek-R1 的 GRPO），但完整训练的计算成本很高。训练动态的几何结构此前未被充分探索。

## 方法

- **发现 1**：RLVR 权重轨迹是低秩的——每个张量的 delta 可以用 rank-1 SVD 近似
- **发现 2**：Rank-1 系数接近线性演化（R² > 0.98）
- **RELEX 算法**：早期 checkpoint 的 delta → SVD rank-1 子空间 → 线性拟合 → 外推
- **零训练成本**：闭式截断 SVD + 两参数最小二乘拟合

## 关键结果

### In-domain MATH（观察 15-20%，外推到 500 步）

| 模型 | Base | Full RLVR | RELEX |
|------|:----:|:---------:|:-----:|
| Qwen2.5-Math-1.5B | 48.2 | 71.5 | **71.6** |
| Qwen3-4B-Base | 64.0 | 85.5 | **85.6** |
| Qwen3-8B-Base | 73.9 | 88.5 | 87.4 |

### OOD 基准（5 个 benchmark 平均）

| 模型 | Full RLVR | RELEX |
|------|:---------:|:-----:|
| Qwen2.5-Math-1.5B | 28.4 | **30.0** |
| Qwen3-4B-Base | 42.3 | **43.0** |
| Qwen3-8B-Base | 47.1 | 46.2 |

### vs 基线（15-20% 成本）

RELEX 优于 ExPO、AlphaRL、Logits Extrapolation 和 Weight Extrapolation。

### 外推极限

- 观察 50 步，可外推到 1000 步并持续改进
- 超过观察窗口 10-20 倍仍保持稳定

## 亮点

- 极致的简洁性：rank-1 + 线性拟合就足够了；rank-5/10 没有额外收益
- 有价值的理论洞察：RLVR 优化沿着直线轨迹进行
- 实用性：开源代码 + HuggingFace checkpoint 已发布
- 去噪效果：rank-1 投影丢弃了优化噪声

## 局限

- 仅在数学推理任务（MATH/AIME/AMC）上验证，未涉及代码、Agent 或通用对话
- 需要已有 RLVR 训练轨迹作为起点
- 观察窗口的选择较为敏感

## 个人评价

最令人兴奋的是反直觉的发现：大家都认为 RLVR 把模型推向了复杂的高维空间，但实际轨迹几乎是一条直线。这可能意味着 Agent RL 训练（比如 GWS 路径规划 RL）也有类似的低秩结构。

去噪视角对 GWS 奖励曲线分析也很有启发。
