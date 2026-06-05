# When Denser Credit Is Not Enough: Evidence-Calibrated Policy Optimization for Long-Horizon LLM Agent Training

**arXiv**: 2606.05885 | **GitHub**: 无 | **机构**: Shanghai Jiao Tong University (上海交通大学)

---

## 一句话总结

ECPO 通过对 step-level credit 做 shrinkage calibration 和 variance gating, 解决了 GiGPO 在长 horizon agent 训练中的 divergent anchor bias 问题, 在 ALFWorld 和 WebShop 上分别比 GiGPO 提升 +5.2% 和 +7.3%, 且额外计算开销仅 0.1%.

## 问题背景

长 horizon LLM agent 训练需要 RL 方法对中间决策做 credit assignment. 现有的 group-based 方法(如 GiGPO)通过在 anchor state 上构建 step-level advantage 来提供更密集的 credit signal. 但论文发现, 密集不等于可靠: 在有限 rollout 下, 稀有但幸运的动作会被过度奖励, 产生 divergent anchor bias, 导致训练后期 reward 剧烈振荡(sigma 从 0.555 升至 0.746).

## 核心方法

- **Evidence-Calibrated Action Advantage (ECA)**: 对同一 anchor state 下各 canonical action 的经验回报做 shrinkage estimation, 将低样本量 action 的估计向整体均值收缩, 公式为 mu_tilde = (n * G_bar + kappa * mu) / (n + kappa), kappa 默认 2.0
- **Variance-Gated Credit Weighting (VarGate)**: 将 anchor-level 方差分解为 between-action variance(信号)和 within-action variance(噪声), 只有信号占比高时才使用 step-level credit, 否则回退到 trajectory-level
- **组合优势**: 最终 advantage = A_GRPO + omega * rho_VG(s) * A_act(s, a), 使用 PPO-style clipped objective 优化
- **Critic-free**: 不需要额外的 critic 网络, 只在 advantage 计算阶段做校准

## 关键结果

| 方法 | ALFWorld Success | WebShop Score | WebShop Success |
|------|---------------------|---------------|---------------------|
| GRPO | 83.9 | 77.1 | 57.3 |
| GiGPO | 87.5 | 80.4 | 64.6 |
| **ECPO** | **92.7** | **84.4** | **71.9** |

- Qwen2.5-1.5B-Instruct 上, ECPO 比 GiGPO 在 ALFWorld +5.2%, WebShop +7.3%
- Qwen2.5-7B-Instruct 上, ECPO 在 ALFWorld 91.9% vs GiGPO 90.8%, WebShop 74.7% vs 72.4%
- 训练稳定性: reward 标准差从 GiGPO 的 0.746 降至 0.555
- 计算开销: advantage 计算阶段仅增加 0.1%

## 亮点

- 发现了 group-based RL 中一个被忽视的根本问题: credit density 不等于 credit reliability
- ECA 和 VarGate 两个组件独立有效(+2.3% 和 +2.1%), 组合后效果叠加(+5.2%), 说明互补性好
- 方法极简: 不需要 critic, 不改模型结构, 只在 advantage 计算阶段加 shrinkage 和 gating
- 在不同 rollout group size (N=4,8,10) 下都稳定优于 GiGPO

## 局限

- 只在 ALFWorld 和 WebShop 两个 benchmark 上验证, 都是相对结构化的环境
- 没有开源代码
- canonical action 的设计依赖环境提供的 canonicalizer, 泛化到更开放的 agent 场景(如 web browsing, coding)尚不清楚
- 与更复杂的 agent RL 方法(如 ARPO, EPO)没有对比

## 个人评价

这篇论文的核心洞察很精准: 在 agent RL 训练中, 不是 credit 越密越好, 而是要看 credit 是否可靠. 这和统计学里 James-Stein estimator 的精神一致, 小样本估计需要向总体均值收缩. 方法设计也很干净, ECA 做 shrinkage, VarGate 做方差分解, 两个正交的校准手段组合使用. 对于做 agent RL 的人来说, 这个问题(GiGPO 的 divergent anchor bias)是真实存在的, ECPO 提供了一个 practical 的解决方案. 不过实验规模偏小, 只有 1.5B 和 7B 两个模型, 两个 benchmark, 如果能在更复杂的 agent 任务(如 SWE-bench, WebArena)上验证会更有说服力.
