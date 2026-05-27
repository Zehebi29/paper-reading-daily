# Hera: Learning Long-Horizon Coordination for Device-Cloud Collaborative LLM Agents

**arXiv**: [2605.24598](https://arxiv.org/abs/2605.24598) | **GitHub**: 无 | **机构**: 未明确标注（arXiv cs.AI / cs.MA）

---

## 一句话总结

Hera 是一个 step-level 的端云 LLM Agent 协调器，通过模仿学习冷启动 + 成本感知强化学习两阶段训练，在 ALFWorld / WebShop / AppWorld 三个长程 Agent 基准上达到 cloud-only 92.5% 的成功率，同时只在 46.3% 的步骤调用云端。

## 问题背景

LLM Agent 部署面临端云困境：设备端模型推理快、成本低，但推理能力弱；云端模型能力强，但延迟高、费用贵。现有路由方法通常做粗粒度的 task-level 决策（整个任务交给设备或云端），无法适应长程 Agent 任务中步骤级难度的动态变化。论文发现，在设备失败但云端成功的任务中，两者在 34.9%（ALFWorld）和 39.3%（WebShop）的步骤上输出完全一致，意味着理想路由器只需在不到 25% 的步骤调用云端即可匹配 cloud-only 性能。

## 核心方法

- **Step-level 路由**：Hera 是一个约 500M 参数的轻量 decoder-only Transformer + 二分类头，每一步决定调用设备还是云端 Agent
- **Stage 1 - 模仿学习冷启动**：将 step-level 路由建模为监督分类问题，在 cloud 轨迹上回放设备 Agent，设备-云端动作不一致的步骤标记为“调用云端”，用 BCE 损失训练
- **Stage 2 - 成本感知强化学习**：跨轨迹聚合相同状态，对每个状态估计期望任务回报和未来云端调用次数，构建偏好标签（优先选高回报，平局时选低云端成本），用 RL 损失更新路由器，加 L2 正则防止偏离 IL 初始化
- **关键洞察**：step-level 的路由决策之间存在依赖关系（当前步骤的选择影响后续步骤的难度），RL 阶段能学到这种跨步骤的依赖，而纯 IL 不能

## 关键结果

| 方法 | 成功率（相对 cloud-only） | 云端使用比例 |
|------|--------------------------|-------------|
| Device-only | 低 | 0% |
| Cloud-only | 100%（基准） | 100% |
| Random routing | ~50% | 50% |
| RouteLLM（最强基线） | 86.2% | 62.1% |
| **Hera** | **92.5%** | **46.3%** |

- 三个基准平均：ALFWorld、WebShop、AppWorld
- IL 单独已超过 random routing；RL 在此基础上再提升成功率 +6.6%，同时减少云端调用 -2.6 次/任务
- RL 单独（无 IL 冷启动）效果差，说明两阶段都不可或缺
- 额外计算开销约 61ms/step，可忽略
- 跨模型对泛化：Qwen2.5-7B + Qwen-Max、Llama 3.1 8B + GPT-5.4 均有效

## 亮点

- 首个针对 LLM Agent 的 step-level 端云协调器，粒度比现有 task-level 路由细得多
- 两阶段训练设计合理：IL 提供稳定初始化，RL 学到跨步骤依赖并优化成本-性能 Pareto 前沿
- 实验覆盖三个不同类型的长程 Agent 基准（embodied / web navigation / coding），泛化性强
- 分析部分有洞察：设备-云端 agreement 与推理长度、预测熵、轨迹位置的相关性

## 局限

- 假设同步、始终在线的设备-云端连接，不适用于离线或高延迟场景
- 没有隐私保护机制：需要将设备端状态发送到云端做决策
- 仅支持单设备-单云端拓扑，未扩展到多设备或多云端
- 没有开源代码

## 个人评价

这篇论文的核心想法很实用：在 Agent 任务中，大部分步骤其实设备端就能搞定，只有少数关键步骤需要云端“救场”。两阶段训练的思路（先 IL 冷启动再 RL 微调）在 Agent RL 领域越来越常见，和之前读的 SkillOpt、CopT 等论文的训练范式类似。不过 Hera 专注于 routing 而非 policy 本身，是一个正交的优化维度。

启发：如果把这种 step-level routing 的思路和 Agent policy RL 训练结合起来——比如训练一个“知道自己哪里弱”的 Agent，只在不确定时才请求外部帮助——可能会产生更实用的系统。另外，成本感知 RL 的偏好标签构建方式（回报优先、成本平局决胜）值得在其他资源受限场景借鉴。
