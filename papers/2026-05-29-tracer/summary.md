# TRACER：用后悔匹配和内部信用分配实现多LLM协作推理

**arXiv**: 2605.28699 | **GitHub**: https://github.com/Shark-Forest/TRACER | **机构**: 未明确标注（论文中无机构信息）

---

## 一句话总结

TRACER 提出一个两层强化学习框架，让多个 LLM 学会“什么时候该说话”和“该说什么”，在 GSM8K/MATH500/GPQA-Diamond 上用仅 3 次调用和约 1000 token 就达到了接近 Self-Consistency 的准确率，推理成本降低了一个数量级。

## 问题背景

把单 agent 的 RL（如 GRPO）直接搬到多 agent 场景会遇到三个麻烦：（1）奖励稀疏、角色搭便风、训练开销大；（2）agent 只是模仿协作而非真正学会协作；（3）固定的交互协议（辩论、投票）容易陷入局部最优且训练不稳定。核心问题是：多 agent 协作中，“谁在什么时候说话”和“每次说什么”这两个决策耦合在一起，导致信用分配极其困难。

## 核心方法

- **两层分离架构**：将协作决策拆成 Controller-Regret Layer（学“何时说话”）和 Generation-Credit Layer（学“说什么”），解耦两个维度
- **后悔匹配控制器**：每个 agent 有一个二元动作空间 {speak, skip}，通过 counterfactual regret minimization 学习最优发言策略，基于投票得分桶和迭代轮次的离散状态
- **角色特异性 GSPO**：Proposer 和 Reviewer 各自接收独立奖励信号——Proposer 看答案对不对，Reviewer 看判断对不对——避免搭便风
- **只有控制器采样的动作进入训练轨迹**，候选生成仅用于参数更新不进入推理链路，大幅降低计算开销

## 关键结果

### Qwen2.5-7B-Instruct 准确率

| 方法 | GSM8K | MATH500 | GPQA-D | Tokens/任务 | 调用次数 |
|---|---|---|---|---|---|
| CoT | 91.60% | 75.50% | 36.40% | 1355 | 1.00 |
| Self-Consistency (10路) | 92.40% | 67.60% | 27.27% | 13551 | 10.00 |
| Single-Agent GRPO | 88.25% | 47.80% | 3.41% | 252 | 1.00 |
| MAPoRL | 83.85% | 10.00% | 20.71% | 2099 | 4.00 |
| **TRACER** | **89.01%** | **61.20%** | **35.35%** | **1014** | **3.02** |

### 消融实验关键发现

- 移除 Reviewer 组件导致准确率下降 6.6 个百分点、token 消耗增加 1200
- 移除 Proposer 控制器导致准确率下降 7.5 个百分点
- 投票状态和迭代状态都对性能有显著贡献

## 亮点

- 将博弈论中的后悔匹配引入多 agent LLM 训练，理论上有收敛保证（Theorem 1），设计很优雅
- 推理成本极低：平均仅 3 次调用、约 1000 token，相比 Self-Consistency 的 13000+ token 便宜了十几倍
- 在 GSM8K 上训练、MATH500 和 GPQA-Diamond 上零样本迁移效果不错，说明学到的是通用协作策略而非过拟合
- Reviewer 的作用被消融实验强力验证——多 agent 协作中“会挑毛病”比“会说话”更关键

## 局限

- 实验仅在数学推理任务上验证（GSM8K/MATH500/GPQA-Diamond），没有在 coding、agent benchmark（如 SWE-bench）等场景测试
- 最大 backbone 是 7B 模型，更大模型上的效果未知
- 控制器的状态空间设计（投票桶 + 迭代轮次）较为简单，可能限制了在更复杂交互场景中的泛化
- 论文没有报告训练时间和计算开销，multi-agent RL 训练通常比 single-agent 贵很多

## 个人评价

这篇论文的两层解耦设计是个不错的思路——“谁说话”和“说什么”分开优化，避免了 multi-agent RL 中常见的信用分配混乱。后悔匹配的选择也很有意思，因为它本身就是为合作博弈设计的，比直接用 PPO/GRPO 硬搬到 multi-agent 更自然。不过实验场景偏窄，只在数学推理上跑了，如果能在 agent 任务（如 web browsing、tool use）上也验证一下，说服力会强很多。另外 1000 token 的推理成本确实很有吸引力，如果这个优势能迁移到更复杂的任务，对实际部署很有价值。

从技术角度看，Reviewer 比 Proposer 更重要的消融结果很值得深思：在 multi-agent 协作中，“会批评”可能比“会提议”更关键。这对设计 multi-agent 系统有直接启发——与其堆更多 Proposer，不如确保有一个靠谱的 Reviewer。
