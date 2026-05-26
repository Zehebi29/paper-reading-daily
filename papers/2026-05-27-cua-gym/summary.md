# CUA-Gym: Scaling Verifiable Training Environments and Tasks for Computer-Use Agents

**arXiv**: 2605.25624 | **GitHub**: https://github.com/xlang-ai/CUA-Gym | **机构**: XLang Lab @ UCSD / Qwen (Alibaba) / 清华

---

## 一句话总结

CUA-Gym 提出一套可扩展的 RLVR 数据生成管线，用生成器-判别器对抗循环自动产出 (任务, 环境, 奖励函数) 三元组，训练出的 3B 和 17B 模型在 OSWorld-Verified 上分别达到 62.1% 和 72.6%，且 3B 模型仅用 1/10 参数就追平了 397B 基线的水平。

## 问题背景

RLVR（可验证奖励的强化学习）在数学、代码、工具使用等领域已经证明了威力，但在 Computer-Use Agent (CUA) 上始终推不动。核心瓶颈在于：每个 RLVR 训练样本需要同时满足“任务指令一致、环境可执行、奖励函数可验证”三个条件，人工构造覆盖窄，LLM-as-Judge 构造又缺乏确定性验证。这两头都卡住，RLVR 在 CUA 上的规模化就没法实现。

## 核心方法

1. **对抗式生成-判别循环**：Generator 代理写环境构建脚本，Discriminator 代理（在信息屏障隔离下）仅凭任务描述写奖励函数，两者通过 Orchestrator 协调迭代，直到五个一致性条件全部满足（如 golden 状态奖励=1.0、初始状态奖励=0.0、无违禁模式等）。

2. **信息屏障设计**：Discriminator 无权访问 Generator 的脚本和工作目录，强制奖励函数必须从语义层面验证任务完成度，杜绝 reward hacking。

3. **CUA-Gym-Hub 环境合成**：基于 O*NET 职业分类体系和 Anthropic 经济指数，用多代理管线（Plan Agent → Dev Agent → Web Agent）自动生成 94 个高保真 Mock Web 应用，每个 Mock 支持数千个独立任务状态。

4. **GSPO 训练算法**：在 GRPO 基础上加入 group-based superiority 比较，配合 KL 散度约束，在 32,112 个验证过的 RLVR 样本上训练。

## 关键结果

| 模型 | OSWorld-Verified | WebArena（跨域迁移） |
|:---|:---:|:---:|
| CUA-Gym-A3B | **62.1%** (+7.6pp) | 44.5% |
| CUA-Gym-A17B | **72.6%** (+10.4pp) | 56.0% |
| 397B 基线（未改） | ~62% | — |

额外发现：数据量与环境多样性是互补轴，单靠轨迹量无法替代多样化的环境暴露。RL 训练后模型自发学会每回合发出多个工具调用，轨迹长度缩短 33-45%，性能不变。WebArena 上的提升说明学到的是通用 CUA 能力。

## 亮点

- 端到端可复现：完整开源管线、数据集、Mock 环境、模型权重
- 信息屏障机制优雅：用对抗式博弈代替人工标注，自动保证奖励函数质量
- 3B 模型 1/10 参数追平 397B：说明 RLVR 训练是让模型更高效利用已有能力
- 跨域迁移成立：验证了学到的是通用 CUA 能力，不是刷榜技巧

## 局限

- Mock 环境不完整：缺少认证、三方集成、网络延迟、服务端异常等真实条件
- Generator-Discriminator 循环成本高，算力开销不小
- 仅覆盖 GUI 类操作，对终端、API、文件系统等非 GUI 界面不适用
- Qwen 系模型为主，管线对基座模型的选择有否依赖尚未充分讨论

## 个人评价

这篇最打动我的不是 72.6% 的 SOTA 数字，而是它精准抓住了 RLVR 在 CUA 领域“卡住”的结构性原因——不是算法不够好，是数据管线根本建不起来。CUA-Gym 用“Agent 生成 Agent 的训练数据”这个元思路，把 Generator-Discriminator 对抗循环做到工业级可靠，搭配自动合成的 Mock 应用生态，一下把 CUA RLVR 数据规模从“几百条”推到“三万条”。

对做 GWS 的启发：我们在焊缝路径规划中也会遇到类似的“高质量训练数据稀缺”问题——手工标注路径太贵，仿真生成又不够逼真。CUA-Gym 的对抗式数据质量保证思路值得借鉴：也许可以用一个“路径规划器”Agent 生成焊接轨迹，另一个“质量检查器”Agent（不看规划过程、只看最终焊缝质量）来验证，通过迭代循环自动积累高质量训练数据。
