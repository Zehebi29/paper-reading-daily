# AgentJet: A Flexible Swarm Training Framework for Agentic Reinforcement Learning

**arXiv**: 2606.04484 | **GitHub**: https://github.com/modelscope/AgentJet | **机构**: Alibaba Group (Tongyi Lab)

---

## 一句话总结

AgentJet 是阿里通义实验室提出的分布式 swarm 训练框架，通过将模型优化与 agent 执行完全解耦，支持异构多模型 RL 训练、多任务混合训练和热插拔调试，并通过 timeline merging 实现 1.5-10x 训练加速。

## 问题背景

现有 LLM agent RL 训练框架（如 OpenRLHF、veRL）采用中心化架构，将 agent rollout 和模型优化耦合在同一 GPU 集群上。Agent 环境交互通常是 CPU-intensive 的，占用 GPU 资源；无法支持多个异构模型同时训练；不同任务的 agent runtime 可能互斥，难以混合训练；代码迭代需要重启整个训练流程。

## 核心方法

- **Swarm 架构解耦**: Server 节点只负责模型优化 (GPU), Client 节点只负责 agent 执行和环境交互 (CPU), 通过 OpenAI-compatible API 通信
- **Timeline Merging**: 将多轮 agent 交互中的冗余上下文（重复的 system prompt、tool definitions、observation history）合并，实现 1.5-10x 训练加速
- **异构多模型 RL**: 同时训练多个不同规格的 LLM（如 planning 用 14B、execution 用 7B），参数不共享
- **多任务混合训练 (Cocktail Training)**: 同时训练不同类型任务（如 AppWorld 工具调用 + AIME 数学推理），用隔离的 agent runtime
- **热插拔调试**: 训练过程中可替换 Client 节点代码，无需重启 Server

## 关键结果

| 实验 | 模型 | 基线 | 训练后 | 备注 |
|:---|:---|:---|:---|:---|
| Werewolves (shared param) | 7B | 23.0% | 47.2% | 狼人杀胜率 |
| Werewolves (shared param) | 14B | 40.9% | 64.7% | 狼人杀胜率 |
| Werewolves (non-shared param) | 3x14B-LoRA | - | 66.5% | 独立参数更高 |
| AIME (cocktail) | Qwen3-8B | - | 0.72 reward | 专用训练 0.73 |
| AppWorld (cocktail) | Qwen3-8B | - | ~10pt gap | 比专用训练低 |
| 框架无关性验证 | - | - | cross-arm gap <0.04 | 4种 agent 框架一致 |
| Timeline merging 加速 | - | 2160s | ~3x 提速 | 1.5-10x depending on task |

## 亮点

- **解决了真实痛点**: 中心化架构的 GPU-CPU 耦合问题是目前 agent RL 训练的主要瓶颈，swarm 架构的解耦设计很自然
- **生产友好**: 支持任意基于 OpenAI SDK 的 agent 框架，实际验证了 4种框架训练动力学一致
- **实用的加速**: Timeline merging 解决了多轮交互中上下文膨胀的实际问题
- **异构多模型训练**: 第一次在 agent RL 中实现了真正的参数不共享多模型 MARL

## 局限

- **Cocktail training 有损失**: AppWorld 任务比专用训练低约 10 个点，梯度稀释问题未完全解决
- **实验仅在狼人杀和 AppWorld 上验证**: 缺乏更广泛的 benchmark 对比（如 WebArena、SWE-bench）
- **未提供大规模训练的 scaling 数据**: 实验主要用 7B/14B 模型，未验证更大规模

## 个人评价

AgentJet 是目前我见过的最实用的 agent RL 训练框架之一。它解决的不是算法创新问题，而是工程架构问题——怎么让 agent RL 训练在实际生产环境中跑起来。Swarm 架构的解耦设计很简洁，允许用户在笔记本电脑上跑 agent，在 GPU 集群上跑训练，这对研究者很友好。Timeline merging 是个实用的优化，解决了多轮 agent 交互中上下文膨胀的真实痛点。

不过，这是一个偏工程的贡献，算法层面还是用的 GRPO。对于研究者来说，它的价值在于提供了一个可以快速实验新算法的平台，而不是训练方法本身的突破。GitHub 已开源，值得关注其后续的社区发展。
