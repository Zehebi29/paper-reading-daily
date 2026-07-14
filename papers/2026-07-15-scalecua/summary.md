# ScaleCUA: Scaling Computer Use Agents with Verifiable Task Synthesis and Efficient Online RL

**arXiv**: 2607.11185 | **GitHub**: https://github.com/THUDM/SCALE-CUA | **机构**: Tsinghua (THUDM)

---

## 一句话总结

ScaleCUA 通过自动化生成可验证的 GUI 任务 + 在线 RL 的方式，在 OSWorld 上达到 68.7% （开源 SOTA），打破了可验证数据缀乏和在线 RL 低效的瓶颈。

## 问题背景

Computer use agents (CUAs) 是通过视觉感知和 GUI 操作来自动化数字工作流的强大接口。在线可验证奖励 RL (RLVR) 是提升其能力的关键方向，但受到两个核心瓶颈限制：(1) 可验证的培训数据极度缀乏，从头构建需要大量人工；(2) 多轮 GUI 交互的在线 RL 训练效率低下。

## 核心方法

1. **VeriGen**: 自动化生成可验证 RL 任务的管道，通过迭代 docker 交互 + 多智能体反馈循环，生成 24K+ 可验证任务和近 3K 高质量 RL 任务。
2. **Frontier Sampling**: 跟踪每个任务的当前能力水平，动态分配滚动资源到学习前沿任务，提升样本效率。
3. **Visual Context Segmentation (VCS)**: 滑动窗口机制平衡滚动和训练引擎压力，实现 2.83x 训练加速。
4. 采用 GRPO + 可验证奖励进行多轮在线 RL 训练。

## 关键结果

| 基准 | ScaleCUA | 之前开源 SOTA | 提升 |
|:---|:---:|:---:|:---:|
| OSWorld | 68.7% | ~60% (CUA-Gym) | +8.7pp |
| ScienceBoard | 54.0% | - | 新 SOTA |
| 训练速度 | 2.83x 加速 (VCS) | 基线 | - |
| 生成任务量 | 24K+ 可验证 / ~3K RL | 手动任务 | 规模化 |

## 亮点

1. 完整的从数据生成到在线 RL 的闭环管道，单一框架解决数据缀乏和训练效率两大问题。
2. VeriGen 的多智能体反馈循环设计很巧妙，将任务生成可信任化。
3. Frontier Sampling 是一个简单但有效的策略，将资源聚焦在当前能力边界上。
4. 全开源 (code + models + datasets) ，具备生产实践价值。

## 局限

1. 仅局限于可验证奖励的任务类型，无法覆盖更开放的场景。
2. 任务生成依赖于强大的基础模型 (GPT-5级别)，成本较高。
3. 在 ScienceBoard 上的结果尚未与商业闭源系统对比。

## 个人评价

这篇论文的最大亮点是“解决了一个真实的工程问题”——可验证任务的缀乏。VeriGen 的多智能体管道设计很值得学习：用 Generator + Discriminator 的多智能体反馈循环保证任务质量，而且 100+ 并发 worker 的设计让规模化变得可行。对于我们做 GWS 的 Agent 行为数据生成来说，这种“自动化可验证任务生成 + 在线 RL”的模式很有启发，特别是 Frontier Sampling 的资源分配策略，可以直接应用到我们的 GWS 训练中。
