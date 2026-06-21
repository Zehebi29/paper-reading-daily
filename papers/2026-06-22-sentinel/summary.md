# SENTINEL: Failure-Driven Reinforcement Learning for Training Tool-Using Language Model Agents

**arXiv**: 2606.12908 | **GitHub**: 无 | **机构**: Ziyi Wang, Yuxuan Lu, Yimeng Zhang, Qun Liu, Chen Luo, Jiri Gesi, Hanqing Lu, Yisi Sang, Manling Li, Jing Huang, Dakuo Wang

---

## 一句话总结

SENTINEL 通过分析 agent 自己的失败轨迹来动态生成针对性训练任务, 在 Tau2-Bench Retail 上以 Qwen3-4B 为底座实现 Pass^1 从 66.4 提升至 74.9, 且在 SFT 强模型上通用 RL 会导致性能崩塓 (74.3→68.1) 而 SENTINEL 能继续提升 (74.3→78.1)。

## 问题背景

用 RL 训练工具使用型 LLM agent 时, 训练任务分布通常是静态预定义的。随着策略迭代, 许多任务变得对当前策略不再有信息量（太简单、太难或重复）, 导致大量 rollout 被浪费。如何让训练任务自适应地跟上策略的演进, 是一个关键问题。

## 核心方法

1. **Controller-Proposer-Solver 三角循环**: Controller 分析失败轨迹提取错误模式, Proposer 根据指令生成针对性任务, Solver 在新任务上训练 RL
2. **轨迹驱动的任务构建**: Proposer 先在环境中构建可执行的工具调用轨迹, 再将其改写为自然语言任务, 确保任务可执行且自带参考答案
3. **失败模式分类**: 系统识别四类错误 — 缺失必要动作、错误参数选择、多记录混淆、执行控制失误
4. **奖励塑形**: 除了终端任务成功奖励, 加入动作级检查和退化惩罚（重复工具调用、格式错误等）
5. **GRPO 优化**: 基于 Group Relative Policy Optimization 训练, 在原始任务 + 失败针对任务的混合缓冲区上优化

## 关键结果

**Tau2-Bench Retail (Qwen3-4B)**:

| 方法 | Pass^1 | Pass^2 | Pass^3 |
|:---|:---|:---|:---|
| Base | 66.4 | 51.6 | 43.2 |
| General RL | 69.0 | 55.6 | 47.4 |
| **SENTINEL** | **74.9** | **60.5** | **51.2** |

**SFT + RL 实验**:

| 方法 | Pass^1 |
|:---|:---|
| SFT | 74.3 |
| SFT + General RL | 68.1 (↓ 崩塓) |
| SFT + SENTINEL | **78.1** (↑) |

核心发现: 通用 RL 在强模型上会导致性能崩塓, 而 SENTINEL 的失败驱动方法能持续提升强模型。

## 亮点

1. 用自身失败作为训练信号的设计很直观且有效, 类似 curriculum learning 但更自动
2. SFT+General RL 性能崩塓的发现很重要 — 说明对强模型做 RL 需要更精细的任务设计
3. 失败模式随训练演进而变化（早期是缺失动作, 后期是策略级错误）, 说明系统真在自适应
4. 奖励设计考虑周全, 包含动作级检查而不只是终端奖励

## 局限

1. 实验仅在 Tau2-Bench Retail 一个域上验证, 泛化性待测试
2. 没有公开代码/工具, 复现成本不明
3. 失败模式分析依赖 LLM 自身能力, 小模型可能难以准确诊断复杂失败

## 个人评价

这篇论文的核心贡献是揭示了一个很实用的规律: 对强模型做 RL 时, 随机任务不仅无用还有害。General RL 从 74.3 跌到 68.1 这个结果很关键 — 它说明现有的“用更多任务做 RL”范式在强基座模型上是危险的。SENTINEL 的解法 — 让模型告诉你它在哪里失败 — 简单优雅且有效。不过实验太局限于一个域, 且没有代码, 让人怀疑它能否接地气地接到更广泛的 agent 场景。对于做 agentic RL 的人来说, “强模型+RL会崩塓”这个观察值得记住, 可能比方法本身更有启发。
