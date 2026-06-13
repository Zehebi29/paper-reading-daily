# Orch-RM: Multi-Agent Orchestration Reward Modeling

**arXiv**: 2606.13598 | **GitHub**: https://github.com/Wang-ML-Lab/OrchRM (not yet public) | **机构**: Rutgers University, Salesforce Research

---

## 一句话总结

Orch-RM 在编排层级（planning stage）而非轨迹层级（full execution）训练 reward model, 用 2.38M token 达到传统方法 142.80M token 的效果, BrowseComp+ 准确率提升 8%, AIME 24&25 提升 3%.

## 问题背景

Multi-Agent System (MAS) 的 orchestrator 训练面临两个瓶颈: (1) 需要执行完整子 agent 轨迹才能获取 reward signal, 计算成本极高; (2) 人工标注 orchestration 质量不可扩展. 现有方法要么依赖昂贵的 sub-agent rollout, 要么使用 LLM-as-judge 逐轨迹评估, token 消耗巨大.

## 核心方法

1. **编排层级 Reward Model (Orch-RM)**: 不执行子 agent, 直接在 orchestration plan 层面评估质量, 跳过昂贵的 full trajectory execution
2. **自监督数据构建**: 两种偏好对来源 — (a) 训练后 orchestrator vs base model 的编排对比 (Specialized over Base), (b) 正确 vs 错误的编排轨迹对比 (Correct over Incorrect), 混合比 3:1 最优
3. **Bradley-Terry Reward Model**: 基于 Skywork-Reward-LLaMA-3.1-8B 微调, 用 logistic loss 优化偏好对
4. **两大应用场景**: (a) Test-time scaling — Best-of-N 选择, 在子 agent 执行前就筛出最优编排; (b) Orchestrator 训练 — 为 GRPO 提供编排层级 reward signal, 支持从头训练和继续训练

## 关键结果

| 方法 | AIME 24&25 | BrowseComp+ | HotpotQA | GPQA |
|------|-----------|-------------|----------|------|
| **Orch-RM (Best-of-N, N=8)** | **68.33%** | **14.00%** | 42.50% | 66.16% |
| Maj. Vote | 63.33% | 9.50% | 42.50% | 63.63% |
| Oracle (UB) | 76.67% | 24.50% | 57.00% | 84.85% |

效率对比 (BrowseComp+):
- Orch-RM: **2.38M tokens**
- Trajectory-level LLM judge: **142.80M tokens**
- 节省约 **60x** token 消耗

继续训练实验: AIME 24&25 准确率 +3%, token 用量减少 ~10x
从头训练: 接近 trajectory-level RL 方法的性能

## 亮点

1. **编排层级 vs 轨迹层级**: 核心创新在于跳过子 agent 执行, 直接评估 orchestration plan, 计算效率提升数十倍
2. **自监督构建**: 不需要人工标注, 利用已有 orchestrator 的执行记录构建偏好对
3. **双重应用**: 既可用于 test-time scaling (Best-of-N), 也可用于 orchestrator 训练 (GRPO reward), 一套 reward model 两种用途
4. **跨领域泛化**: 数学推理 (AIME)、Web QA (BrowseComp+)、多跳推理 (HotpotQA)、科学问答 (GPQA) 四个领域一致有效

## 局限

1. **数据依赖**: reward model 质量取决于已有 orchestrator 的多样性和质量, 如果初始 orchestrator 太弱则偏好对信息量不足
2. **领域特化**: 每个领域需要单独训练 reward model, 统一的跨域 model 仍是开放问题
3. **子 agent 黑箱**: 编排层级方法假设子 agent 行为可预测, 如果子 agent 本身不稳定, 编排评估可能失准

## 个人评价

这篇论文的核心洞察很有价值: 在多智能体系统中, 编排决策的质量可以在不实际执行子 agent 的情况下被评估. 这类似于人类 manager 在分配任务时, 不需要等下属做完就能判断计划是否合理. 从技术角度看, 把 reward model 从 trajectory level 提升到 orchestration level 是一个自然但重要的抽象, 带来了 60x 的效率提升.

不过需要注意, 这是 preprint (work in progress), GitHub 仓库还未上线. 论文中 orchestrator 使用 Qwen2.5-7B-Instruct, 子 agent 使用 GPT-OSS-120B, 实验设置比较 heavy. 对于想复现的研究者来说, 等代码公开后再跟进可能更实际. 这个工作对我们的启示是: 在 agent RL 训练中, reward signal 的获取粒度可以比 trajectory 更粗 (orchestration level), 只要能有效区分好坏方案即可.
