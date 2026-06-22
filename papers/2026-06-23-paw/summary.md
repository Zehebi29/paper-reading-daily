# Policy and World Modeling Co-Training for Language Agents

**arXiv**: 2606.02388 | **GitHub**: 无 | **机构**: HKUST (香港科技大学)

---

## 一句话总结

PaW 在标准 RL 训练中加入辅助的世界模型预测损失, 用同一组 rollout 同时训练策略和环境动态模型, 在 ALFWorld/WebShop/Search QA 上一致提升 2-9 个百分点, 且推理时零额外开销.

## 问题背景

RL 训练 LLM agent 时只告诉模型"哪个动作得了高分", 但不教模型"这个动作对环境做了什么". 现有的世界模型方法要么需要单独的模拟器, 要么需要额外训练阶段, 要么在推理时增加计算开销. PaW 的核心观察是: on-policy RL rollout 本身就包含环境动态的监督信号 -- 每个 transition 都配对了一个动作和它导致的下一个观测.

## 核心方法

- **动作熵选择 (Action-Entropy Selection)**: 只选 rollout 中动作熵高的 transition 做 WM 训练, 因为高熵时刻代表策略不确定的关键决策点, 这些 transition 的后果信息量最大
- **Clipped MAE 损失 (CMAE)**: 用 MAE 代替交叉熵做 next-observation 预测, 避免对不可预测的噪声 token (如随机 ID、品牌名) 过拟合; 加置信度裁剪, 模型已经预测准的 token 不再参与梯度
- **奖励自适应损失平衡 (Reward-Adaptive Balancing)**: WM 损失权重 lambda 随 rollout 组的平均回报反比变化 -- 表现差的组加强 WM 监督, 表现好的组聚焦策略优化
- **推理不变**: 训练时多一个辅助损失, 推理时和普通 policy model 完全一样, 不需要模拟器或规划步骤

## 关键结果

| 基准 | 模型 | 基线 RL | RL+PaW | 提升 |
|:---|:---|:---|:---|:---|
| ALFWorld (平均成功率) | Qwen2.5-1.5B | GRPO: 70.0% | GRPO+PaW: 77.9% | +7.9% |
| ALFWorld (平均成功率) | Qwen2.5-1.5B | GIGPO: 87.6% | GIGPO+PaW: 90.4% | +2.8% |
| WebShop (成功率) | Qwen2.5-1.5B | GRPO: 60.6% | GRPO+PaW: 68.6% | +8.0% |
| WebShop (成功率) | Qwen2.5-1.5B | GIGPO: 66.2% | GIGPO+PaW: 75.3% | +9.1% |
| Search QA (平均分) | Qwen2.5-3B | GRPO: 43.9% | GRPO+PaW: 44.8% | +0.9% |
| WebShop (稀疏奖励) | Llama3.2-3B | GRPO: 4.0% | GRPO+PaW: 62.2% | +58.2% |

- PaW 在 4 种 RL 算法 (GRPO/GIGPO/PPO/RLOO) 和多个模型家族 (Qwen 1.5B-14B, Llama 3.2-3B) 上一致有效
- Llama3.2-3B 在 WebShop 上 vanilla GRPO 完全失败 (4.0%), PaW 将其拉到 62.2%, 说明 WM 辅助监督在稀疏奖励场景下是"救命"信号
- 额外计算开销仅约 2.1%

## 亮点

- 观察极其简洁: RL rollout 本身就是世界模型的免费监督, 不需要额外数据或模拟器
- 三个技术组件 (熵选择/CMAE/自适应平衡) 各解决一个具体的噪声问题, 设计有层次
- 跨算法、跨模型、跨任务的一致提升说明方法有普适性
- 推理时零开销, 工程友好

## 局限

- 实验仅在文本环境 (ALFWorld/WebShop/Search QA) 上验证, 未涉及更复杂的 GUI/Web 操作或多轮工具调用
- 世界模型的"预测"仅限于 next-observation token 预测, 没有显式的状态转移建模或规划能力
- 9 页篇幅较短, 消融实验可以更充分 (比如单独验证每个组件的贡献)

## 个人评价

PaW 的核心 insight 非常干净: 你已经在做 RL rollout 了, 那些 transition 里天然就有环境动态的信息, 为什么不顺手学一下? 这和 EnvRL (2606.17680) 的思路有相似之处 -- 都是从 RL 训练过程中提取额外的环境理解信号 -- 但 PaW 的做法更轻量, 不需要单独的预测模型或辅助任务, 直接在同一个 forward pass 里加一个 loss 就行. 对于 agentic RL 的研究者来说, 这是一个即插即用的 baseline trick, 值得在自己的 pipeline 里试一试. 尤其是 Llama3.2-3B 从 4% 到 62.2% 的结果很说明问题: 当奖励信号太稀疏时, WM 监督可以充当"课程学习"的替代品, 帮模型在获得第一个正奖励之前就学会环境的基本规律.
