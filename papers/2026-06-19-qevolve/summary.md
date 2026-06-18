# Q-Evolve: Self-Evolving LLM Agents with In-Distribution Optimization

**arXiv**: 2606.07367 | **GitHub**: 无 (项目主页: qevolve.github.io) | **机构**: Eindhoven University of Technology, University of Liverpool, MIT-IBM Watson AI Lab

---

## 一句话总结

Q-Evolve 通过在同分布内联合训练过程奖励 critic 和策略, 解决了 LLM agent 长程稀疏奖励下的信用分配问题, 在 ALFWorld/WebShop/ScienceWorld 上达到 SOTA, 且仅需 13K 环境步数 (比 QLASS 少 25 倍).

## 问题背景

LLM agent 在交互式长程任务中面临稀疏延迟奖励的挑战. 现有过程奖励模型 (PRM) 依赖人工标注或环境回溯, 且随着策略演化, 训练分布与实际交互分布产生漂移, 导致奖励信号不可靠. 如何在不引入额外标注成本的前提下实现稳定的策略自进化, 是核心难题.

## 核心方法

- **混合数据构建**: 将专家示范与 agent 自采集轨迹合并为混合离线数据集 D = D_expert + D_self, 无需额外环境交互
- **加权隐式 Q 学习 (Weighted IQL)**: 在混合数据上训练 value function V 和 Q function, 通过步数权重 w_t = (t/T + d)*0.5 + 0.5 上采样成功轨迹和后期步骤, 稳化 Bellman 备份
- **GAE 过程奖励**: 仅用环境奖励 r_env 计算 GAE 优势值作为步级过程奖励, 排除辅助奖励 r_aux 以保持策略与真实任务目标对齐
- **非对称裁剪策略优化 (BPPO)**: 采用 epsilon_low > epsilon_high 的非对称裁剪 (0.8 vs 0.4), 对负优势动作更激进地抑制, 对正更新保守约束, 避免分布漂移
- **自进化循环**: 策略改进 -> 采集新轨迹 -> 更新数据集 -> 重新训练 critic -> 再次优化策略, 迭代 2-3 轮逐步提升

## 关键结果

| 方法 | WebShop | SciW Seen | SciW Unseen | ALFW Seen | ALFW Unseen | 平均 |
|:-----|:-------:|:---------:|:-----------:|:---------:|:-----------:|:----:|
| GPT-4 (ReAct) | 63.2 | 64.8 | 64.4 | 42.9 | 38.1 | 54.7 |
| Reflexion | 64.2 | 60.3 | 64.4 | 45.7 | 55.2 | 58.0 |
| QLASS | 70.3 | 75.3 | 66.4 | 77.9 | 82.8 | 74.5 |
| **Q-Evolve** | **70.5** | **76.3** | **69.7** | **90.7** | **89.6** | **79.4** |

- 样本效率: 仅用 13K 环境步数, 比 QLASS (320K) 少 25 倍
- 跨模型泛化: 在 Llama-2-7B, Llama-3-8B, Qwen2.5-7B 上均有效
- 消融实验: 移除 BPPO (-29.0%), GAE (-13.3%), 加权 IQL (-5.5%), 回溯标注 (-4.3%)

## 亮点

- 统一的过程奖励 + 策略学习框架, 不需要人工标注或环境回溯
- 非对称裁剪是关键设计, 允许对坏动作激进抑制而对好动作保守更新
- 自进化循环中 critic 和策略共同演化, 每轮更新都保持在当前分布内
- 样本效率极高, 13K 步达到 320K 步的效果

## 局限

- 辅助奖励 r_aux 的具体数值 (-0.3, -0.2, -0.1) 需要针对不同环境手动调参
- 自进化循环次数较少 (2-3 轮), 更多轮次的效果未充分探索
- 仅在三个交互式环境上验证, 未覆盖更复杂的工具使用或多轮对话场景
- 目前没有公开代码

## 个人评价

Q-Evolve 的核心洞察很优雅: 过程奖励不需要独立的 reward model, 直接从 critic 的 Bellman backup 中提取 GAE 优势值就够了, 关键是保持数据分布一致. 这比训练单独的 PRM 简单得多, 也避免了 PRM 与策略之间的分布漂移问题. 非对称裁剪的设计 (epsilon_low=0.8, epsilon_high=0.4) 值得关注, 它本质上是在说 对坏动作要狠, 对好动作要稳, 这在 agent RL 中应该是一个通用的 useful inductive bias. 不过 ICML 2026 的接收说明这个方向已经比较成熟了, 后续工作可能会在更复杂的环境 (如 WebArena, SWE-bench) 上做验证.
