# TRACE: Turn-level Reward Assignment via Credit Estimation for Long-Horizon Agents

**arXiv**: 2607.13988 | **GitHub**: 无 | **机构**: Microsoft Research

---

## 一句话总结

TRACE 提出了一种密集信用分配方法，利用冻结参考模型的 log比率作为状态价值，通过 TD 差分为多轮副本的每个工具调用分配 turn-level 奖励，在无需额外 critic/SFT/中间训练的情况下，将 BrowseComp-Plus 上 Qwen3-4B 从 7.2% 提升至 35.6%。

## 问题背景

多轮 agents 在解决复杂任务时需要执行数十或数百次工具调用，而 RLVR（可验证奖励强化学习）只依赖最终结果奖励，导致信用分配困难。失败的轨迹可能包含很多有用动作，成功的轨迹也可能包含冗余动作，但 outcome-only 训练给所有动作分配相同的优势，导致梯度方差随序列长度增大。

## 核心方法

1. **Log比率状态价值**: 在每次工具调用边界获取冻结参考模型对金答案的 log-可能性，经过 log比率变换得到状态价值
2. **TD差分奖励分配**: 对相邻状态价值做渗水差分，得到每个动作的密集奖励
3. **无额外训练**: 不需要 critical rewards 模型、过程标签或任何额外的蓝作流水线工具调用
4. **可缩放 TD**: 单步 log比率 TD 组件可以跨冗余工具调用进行叠加，自然处理长序列
5. **纯 RL 方案**: 不需要 cold-start SFT、中间任务训练或活网络数据

## 关键结果

| Benchmark | 模型 | 基线 | TRACE | 提升 |
|:---|:---|:---:|:---:|:---:|
| BrowseComp-Plus | Qwen3-4B | 7.2% | 35.6% | +28.4pp |
| BrowseComp-Plus | Qwen3-30B-A3B | 8.4% | 42.6% | +34.2pp |
| 开放网络 (Open-web) | Qwen3-4B | - | 转移效果良好 | 跨域通用 |

## 亮点

1. **极端纯粹的方案**: 无需 SFT、中间训练或活网络数据，单纯靠更好的信用分配就实现了巨大提升
2. **惊人的绝对提升**: Qwen3-4B 7.2% → 35.6% (+28pp)，说明信用分配确实是前沿家副本 RL 的瓶颈
3. **无需额外模型**: 不需要任何额外的 evalutor/critic，只用原始 policy 自身的 log-probability
4. **跨域转移**: 封闭网页环境上学到的搜索行为可转移到开放网络场景

## 局限

1. **无开源代码**: 目前没有公开 GitHub repo，无法复现结果
2. **仅测试了 web search 场景**: 是否适用于 SWE-bench/OSWorld 等其他副本型场景尚未验证
3. **需要 gold answer**: log比率计算依赖于 ground-truth 答案，对开放性任务的适用性有限

## 个人评价

这篇文章的核心见解很简单但很有力: 对于多轮副本的 RL 训练，信用分配比奖励函数更重要。很多工作在优化奖励许许设计、采样策略或训练算法，但 TRACE 表明: 将 sparse terminal reward 转化为 dense turn-level reward 就能带来质的飞跃。+28pp 的提升在 RL 训组中是极其少见的，这表明 browse-comp 这类长序列搜索任务下，传统 outcome-only RL 的信用分配确实是严重失灵的。

对我们的启示: 在 GWS 的 RL training pipeline 中，如果也能引入类似的 turn-level/granular 信用分配机制，可能会显著减少正确路径中的冗余动作问题。TRACE 的方法设计清晰且原理简单，值得在我们的实验中引入尝试。
