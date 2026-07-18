# SEED: Self-Evolving On-Policy Distillation for Agentic Reinforcement Learning

**arXiv**: 2607.14777 | **GitHub**: https://github.com/jinyangwu/SEED | **机构**: Tsinghua University + Zhejiang University + CUHK + NTU + Tongji University

---

## 一句话总结

SEED 提出了一种自我演化的“事后技能蒸馏”框架，让 policy 在 RL 训练中同时扮演“行动者”和“分析者”，将已完成的轨迹解析为自然语言技能并蒸馏回 policy，在 ALFWorld 上比 outcome-only RL 提升了14.9~45.9 个百分点。

## 问题背景

长程度 agentic 任务中，outcome-based RL 只提供疏疏的轨迹级奖励，无法指导中间决策的学习。失败的轨迹可能包含有用的局部行为，成功的轨迹中可能有可复用的策略——但 scalar reward 无法标识它们。这就是“督导差距”（supervision gap）：剧情级结果与 token 级策略学习之间缺少细粒度的信号。

## 核心方法

1. **Hindsight-skills SFT 预训练**：收集普通轨迹，为每条轨迹注释一个“事后技能”（自然语言），微调 backbone 让同一模型具备分析轨迹的能力。
2. **Self-Evolving On-Policy Distillation：**在 RL 训练中，当前 policy 同时收集轨迹 + 分析轨迹提取技能，融合到当前变化的行为模式中。
3. **Skill-induced token-level distillation**：在普通上下文和 skill-augmented 上下文下重新评分 actions，将 skill 引起的概率偏移转换为密集的 token 级蒸馏信号。
4. **Joint optimization**：dense distillation signal + outcome-based RL 共同优化，辅助督导始终与当前轨迹分布保持一致。

## 关键结果

| 基准 | 统计量 | SEED | 对比最佳基线 |
|:---|:---|:---:|:---:|
| ALFWorld (Avg.) | 平均成功率 | **91.8** | 84.4 (GRPO+OPSD) |
| Search-based QA (Avg.) | 平均成功率 | **45.7** | 44.6 (OPSD) |
| WebShop (Succ.) | 成功率 | **78.9** | 68.0 (GRPO+OPSD) |
| 增强样本效率 | 对比 outcome-only RL | 显著提升 | - |

*Note: 节目条源自论文 Figure 1 和全文描述。*

## 亮点

1. **自我演化机制：**policy 更新后自动成为下一个 analyzer，hindsight supervision 与 行为模式共同演进，并非静态的离线蒸馏。
2. **密集 token 级督导：**skill-induced 概率偏移转换为密集督导信号，解决疏疏奖励的督导差距问题。
3. **强范化能力：**SEED 在未见场景下也表现稳健，可扩展到视觉 agent 任务。
4. **完整开源：**GitHub (已发布), 项目页面, 模型均已开源。

## 局限

1. 需要额外的 SFT 预训练阶段来开发“分析轨迹”能力，增加了一定的准备成本。
2. Skill 的质量依赖于 backbone 的分析能力，如析器缺乏够好的问题楼解能力可能会引入噪音。
3. 未评估在更复杂的现实世界任务（如 SWE-bench）上的效果，实验聚焦于文本和视觉 agent 基准。

## 个人评价

SEED 的核心创新点很干脆：不是给模型注入预先定义的技能库，而是让 policy 自己在 RL 训练中动态发现和提炼技能。这种“自我演化蒸馏”的思路很聪明——技能的发现和利用始终与 policy 的当前行为水平保持同步，解决了静态蒸馏方法“技能过期”的痛点。

从实验数据看，ALFWorld 上 91.8 vs 84.4 的提升很可观，而且作者还开源了全套代码和模型，对后续工作很有参考价值。美中不足的是没有在更难的编程 agent 任务上评估（如 SWE-bench），这是一个明显的疏漏。
