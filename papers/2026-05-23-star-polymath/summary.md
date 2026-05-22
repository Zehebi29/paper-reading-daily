# STAR-PolyaMath: Multi-Agent Reasoning under Persistent Meta-Strategic Supervision

**arXiv**: 2605.19338 | **GitHub**: https://github.com/Julius-Woo/STAR-PolyaMath | **机构**: Wu, Zhang, Liu et al.

---

## 一句话总结

STAR-PolyaMath 用一个持久的 Meta-Strategist 智能体（跨尝试记忆+元级控制）配合 Python 编排器，把多智能体数学推理推到了8个顶级竞赛基准的 SOTA，AIME/Putnam/HMMT 全部满分。

## 问题背景

前沿模型已经具备解长链条数学竞赛题的知识和能力，但在实际推理中反复出现三个模式性失败：(1) 幻觉积累 —— 错误沿着长解法轨迹传播，辩论和自我验证不够用；(2) 记忆碎片化 —— 过去尝试的信息不保留，系统重复走死胡同；(3) 推理-工具失衡 —— 过度依赖代码执行反而掩盖了数学结构。作者认为缺的不是模型能力，而是持久的元级监督。
## 核心方法

- 三智能体+编排器架构: Reasoner（主解题者，逐会话持久）、Verifier（严格审查者，按步新建）、Meta-Strategist（元监督者，全问题生命周期持久）、Orchestrator（纯Python控制流，永不推理）
- 控制-推理彻底分离: 编排器用Python写死逻辑，不做任何LLM推理；LLM智能体从不控制流程。这从根本上阻止了误差跨层传播
- 双门验证+辩论循环: Verifier对每一步做Goal Gate（目标达成？）和Logic Gate（推理正确？）双门审查，触发Accept/Challenge/Trace-Back/Propose-Replan四种判决
- 持久Meta-Strategist: 保留整个问题生命周期的失败路径、禁止方向、已抢救结果，是唯一有权判决是否重规划的角色
- Pure-Reasoning Mode: 当Meta-Strategist检测到代码执行变成反效果时，可以强制禁用工具，逼迫纯数学分析

## 关键结果

| 基准 | STAR-PolyaMath | 最强基线 (GPT-5.5 xh) |
|:--------|:--------------:|:-------------------------------:|
| AIME 2025 | 100.00% | 96.67% |
| AIME 2026 | 100.00% | - |
| Apex Shortlist 2025 | SOTA | 93.75% |
| Apex 2025 | 93.75% | 80.21% |
| Putnam 2025 | 100.00% | - |
| IMO 2025 | 100.00% | - |
| HMMT February 2026 | 100.00% | - |
| USAMO 2026 | 100.00% | - |

所有智能体默认使用GPT-5.5 at xhigh effort，每个问题独立运行4次，报告平均值。

## 亮点

1. 控制-推理分离的设计哲学非常务实。纯Python编排器既防止了误差扩散，也让调试和审计变得可预测。
2. 持久Meta-Strategist是真正的创新点。跨尝试记忆让系统能真正吃一堑长一智。
3. Pure-Reasoning Mode很聪明。是对LLM过度依赖工具的显式对抗。
4. 实验结果硬核。在2025-2026年最难的数学竞赛上全满分。

## 局限

1. 计算成本极高。GPT-5.5 xhigh + 反复辩论 + 超时重试，单问题成本比单模型高2-3个数量级。
2. 仅限数学竞赛领域。可验证性是体系成立的前提，迁移到开放域agent任务需要大量改造。
3. Meta-Strategist的智慧完全依赖底层模型。基座模型本身的战略上限决定了整个架构的上限。

## 个人评价

这篇论文让我想起AlphaGo的MCTS——不是在模型层面加能力，而是在推理/搜索层面加结构。STAR-PolyaMath的设计对GWS焊缝路径规划研究也有启发：当LLM Agent做多步规划时，确实需要一个轻量级的确定性编排层来兜底流程，而不是让LLM自己决定下一步做什么。

不过靠GPT-5.5做基座+巨额推理成本换来的满分，本质上是堆算力->堆效果的老路。如果换成开源小模型（Qwen3-8B或DeepSeek系列），效果会掉到什么程度？这才是对方法本身更公平的检验。
