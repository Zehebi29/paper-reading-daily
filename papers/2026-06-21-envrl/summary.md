# EnvRL: Learn from Environment Dynamics in Agentic Reinforcement Learning

**arXiv**: 2606.17680 | **GitHub**: 无 | **机构**: Tsinghua University

---

## 一句话总结

EnvRL 在 agentic RL 的主目标之外加入 state prediction 和 inverse dynamics 两个辅助目标，让智能体从自己的交互轨迹中学习环境动力学，在 ALFWorld/WebShop 上平均提升 4-10 个百分点，几乎零额外计算开销。

## 问题背景

Agentic RL 训练 LLM 智能体时，奖励通常是稀疏的结果奖励（成功/失败），轨迹中的丰富环境动力信息（状态转移）被忽略。如果智能体能从交互经验中学习环境如何运作，就能做出更好的决策。

## 核心方法

- 在主 RL 目标（如 GRPO/GiGPO）之外，加入 **State Prediction (SP)** 目标：给定当前状态和动作，预测下一个观测
- 加入 **Inverse Dynamics (ID)** 目标：给定两个连续状态，推断导致转移的动作
- 三个目标联合优化，辅助目标的权重系数采用 **cosine decay**：早期重点学环境动力学，后期逐渐转向奖励优化
- SP 和 ID 复用 RL rollout 的数据，只增加一次 forward pass，几乎零额外开销（每步更新仅多 1.6s）

## 关键结果

| 基线 | ALFWorld | WebShop | 备注 |
|:---|:---|:---|:---|
| GRPO | 72.8% | 56.8% | Qwen2.5-1.5B-Instruct |
| EnvRL-GRPO | **77.4%** | **67.0%** | +4.6 / +10.2 |
| GiGPO | 86.7% | 67.4% | Qwen2.5-1.5B-Instruct |
| EnvRL-GiGPO | **91.8%** | **74.2%** | +5.1 / +6.8 |

- 达到基线同等性能仅需 **~68.5% 的训练步数**
- 成功轨迹的交互轮数和响应长度更短，说明决策更果断
- OOD ALFWorld 任务和 SearchQA 上也有显著提升

## 亮点

1. **极轻量**: 复用 rollout 数据，每步仅多 1.6s，相当于无开销
2. **方法正交**: 与 GRPO/GiGPO 无关，可插卛式应用到任何 agentic RL 算法
3. **维度合理**: SP 在逻辑结构化环境（ALFWorld）更重要，ID 在嘈声观测环境（WebShop）更重要，符合直觉
4. **样本效率**: 68.5% 训练步数即可达到基线性能

## 局限

1. SP/ID 目标设计仅针对文本智能体，未考虑 VLM 场景
2. 权重系数 λ 需要手动调参（1.5B 用 0.2，7B 用 0.1）
3. 只在 ALFWorld 和 WebShop 两个基准上验证，覆盖场景有限

## 个人评价

这篇论文的核心洞察很简洁：agentic RL 的 rollout 轨迹本身就包含了环境动力学信息，为什么不用起来？加两个辅助 loss 就能让智能体建立内部环境模型，这个思路和 PaW（Policy and World Modeling Co-Training）以及很多 world model 工作的关注点一致，但 EnvRL 的实现更简单——不需要训练单独的 world model，直接在 policy model 上加辅助目标。性价比极高：无额外模型，无额外数据，每步多 1.6 秒。

不过实验只在两个基准上做，没有代码开源，且辅助目标的权重调参还是手动的。如果能自动调节 lambda 并在更多场景（如 WebArena/SWE-bench）上验证，说服力会强很多。总体来说，这是一个简单有效的工作，适合作为 agentic RL 的一个实用插件。
