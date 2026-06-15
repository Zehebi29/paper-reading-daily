# HarnessX: A Composable, Adaptive, and Evolvable Agent Harness Foundry

**arXiv**: 2606.14249 | **GitHub**: https://github.com/Darwin-Agent/HarnessX | **机构**: RUC AI Box (中国人民大学)

---

## 一句话总结

HarnessX 提出了一个可组合、可自适应、可进化的 Agent harness 工厂, 通过 AEGIS 多智能体进化引擎自动优化 harness 配置, 在 5 个 benchmark 上平均提升 +14.5% (最高 +44.0%).

## 问题背景

当前 Agent harness (prompt + 工具 + 记忆 + 控制流) 的设计存在三个核心问题: (1) 手工搭建且静态, 每个新模型/任务都要从头写 scaffold; (2) 架构耦合, prompt/工具/重试策略/记忆混在同一代码里, 改一处容易破另一处; (3) 与模型训练脱钩, harness 执行产生的 trajectory 数据被丢弃而不是用于模型训练. 这些问题导致 Agent 性能严重依赖人工经验, 且无法从执行反馈中自动改进.

## 核心方法

- **Harness 组合 (Composition)**: 将 harness 形式化为一等对象 H=(M,C), 其中 C 由处理器 (processor) 列表和槽位资源组成, 通过 8 个生命周期 hook (task_start, step_start, before_model, after_model, before_tool, after_tool, step_end, task_end) 进行插入, 支持替换代数 (substitution algebra)
- **九维分类体系**: 将 harness 设计空间分解为 9 个维度 (模型选择、上下文组装、工具编排、错误恢复、记忆管理、评估信号、安全护栏、多智能体协调、运行时配置), 每个维度独立可替换
- **AEGIS 进化引擎**: 基于 trace 的多智能体进化引擎, 包含 4 阶段 pipeline (trace 收集 -> 诊断分析 -> 变异生成 -> 评估选择), 将 RL 中的信用分配问题映射到 harness 组件级别的改进信号
- **Harness-Model 共同进化**: 通过共享 replay buffer 交错进行 harness 进化和模型 RL 训练, 实现 cross-harness GRPO, 让 harness 改进和模型改进互相促进

## 关键结果

| Benchmark | 基线最佳 | HarnessX 最佳 | 增益 |
|:---|:---|:---|:---|
| ALFWorld | 最弱基线 | +44.0% | 最大增益 |
| GAIA | 中等 | 显著提升 | - |
| WebShop | 中等 | 显著提升 | - |
| tau3-Bench | 接近天花板 | +1.1% | 近天花板仍有提升 |
| SWE-bench Verified | 中等 | 显著提升 | - |

- 平均绝对增益: +14.5% (跨 15 个模型-benchmark 配置)
- 14/15 个配置获得正向提升, 增益与基线强度呈反比 (inverse scaling)
- 测试模型: Claude Sonnet 4.6, GPT-5.4, Qwen3.5-9B
- 进化轮数: 最多 15 轮

## 亮点

- 将 harness 从"一次性工程"变成"可进化实体", 理论框架完整 (形式化 + 代数 + 分类体系)
- AEGIS 引擎设计巧妙, 将 RL 的信用分配思想迁移到 harness 组件级别的诊断和变异
- Inverse scaling pattern 很有价值: 基线越弱的 Agent, harness 进化带来的增益越大, 说明 harness 优化对弱模型的补偿效应最强
- 跨 3 个模型家族验证, 说明方法的通用性不依赖特定模型

## 局限

- 论文声明代码将在未来开源, 但 GitHub 仓库 (Darwin-Agent/HarnessX) 目前已存在, 实际可复现性需进一步确认
- AEGIS 进化引擎依赖多智能体 LLM 调用进行诊断和变异, 计算成本可能较高
- 15 轮进化后的收敛行为和长期稳定性未充分讨论
- 主要在已有 benchmark 上验证, 真实生产环境的 harness 进化效果未知

## 个人评价

这篇论文的核心洞察很到位: Agent 性能不只取决于模型, harness 的设计同样关键 (甚至更关键). 把 harness 形式化为可组合、可进化的对象, 而非一次性代码, 是一个很有前景的方向. AEGIS 引擎的设计思路 -- 用 trace 驱动的多智能体诊断来替代手工调参 -- 对我们做 Agent RL 训练也有启发: 与其只优化模型权重, 不如同时优化模型和它所处的 harness 环境.

不过, 从实际落地角度看, harness 进化的计算成本和工程复杂度可能是个瓶颈. 每轮进化都需要跑完整的 benchmark 评估, 对于大规模部署来说代价不小. 此外, inverse scaling pattern 虽然有趣, 但也暗示了当基线已经很强时 (如 SWE-bench 上的 SOTA agent), harness 进化的边际收益会递减. 这篇论文更适合被看作一个框架性的贡献, 而非即插即用的工具.
