# AgentOPSD：面向智能体强化学习的递归自蒸馏信用分配

**arXiv**: 2608.05987 | **GitHub**: https://github.com/ZethWang/AgentOPSD | **机构**: 清华大学 + 浙江大学 + 美团

---

## 一句话总结

AgentOPSD 把 token 级自蒸馏信号按回合聚合、并在 log-odds 空间做递归贝叶斯信念更新，将稀疏的轨迹级奖励转化为回合级信用信号，在 ALFWorld / WebShop / Search-QA 上全面超过 GRPO 和强自蒸馏基线，Qwen2.5-7B 在 ALFWorld 达到 89.1% 成功率。

## 问题背景

长程多回合智能体任务通常只在整条轨迹结束后才给出可验证奖励，而 GRPO 这类方法把轨迹级 advantage 均匀广播到所有 token 上，无法区分少数决定成败的关键决策与例行操作，轨迹越长问题越严重。近年 OPSD（on-policy self-distillation）用特权上下文提供更密的 token 级监督，但 token 级信号与"环境只在回合边界响应"的交互结构不对齐，且已有方法孤立看待每个回合，没有考虑前面交互积累的证据。

## 核心方法

- 回合级证据聚合：把 token 级 teacher-student log-prob 差 δ_{k,t}（skill 条件化的特权分支 vs 标准分支）在回合内求和，得到回合级证据 e_k，作为贝叶斯证据的近似。
- 递归信念更新：以组平均成功率 R̄ 为初始先验 B0，在 log-odds 空间按 c_k = γ·c_{k-1} + e_k 累积证据（γ 为几何衰减），得到信念状态 B_k = σ(logit(B0) + c_k)。
- 回合信用 = 边际信念修正 ΔB_k = B_k − B_{k-1}：同一局部信号在结果未定时是关键证据，在信念已饱和时则冗余，天然实现"信号 + 历史"的序贯信用。
- 带符号的方向对齐：信念修正方向与 verifier 结果一致（成功轨迹中向上修正）才获得更多信用，仅用幅度会掉 8.6 个点。
- 完全兼容标准策略优化：λ=0.5 加权 reshape 轨迹级 advantage，无需额外 critic、无需额外 rollout；特权 skill（SkillRL SkillBank）仅训练期使用，推理时不依赖。

## 关键结果

| 环境 (Qwen2.5-7B) | AgentOPSD | GRPO | SDAR | StepOPSD |
|---|---|---|---|---|
| ALFWorld Avg SR (%) | **89.1** | 81.2 | 85.9 | 88.4 |
| Search-QA Avg Acc (%) | **49.2** | 42.0 | 49.0 | 48.2 |
| WebShop Score (%) | **90.2** | 80.9 | 89.4 | 87.2 |

- Qwen2.5-3B：ALFWorld 84.4（GRPO 75.0）、Search-QA 46.7（GRPO 36.4）、WebShop Score 90.4（GRPO 79.8）。
- 长程鲁棒性（ALFWorld 7B）：每多一个回合损失的成功点数，AgentOPSD 仅 -0.54，GRPO -2.91，RLSD -3.59，均匀信用方法退化最快。
- 消融（ALFWorld 7B）：完整方法 89.1；改 token 级累积 85.9；用原始局部 gap 替代递归修正 82.8；丢弃符号只留幅度 80.5；去掉经验先验 B0 锚定 78.9。
- 超参：λ=0.5 最优（更小则 84.4/85.9/83.6），证据衰减 γ 在 0.8~1.0 间变化仅几个点。

## 亮点

- 概念优雅：把"回合信用 = 局部信号对成功信念的边际修正"形式化为贝叶斯递归更新，一个公式同时解释聚合、历史依赖和饱和效应，还带理论附录支撑。
- 轻量实用：无 critic、无额外 rollout、无推理期开销，直接叠加在 GRPO 上即可用，工程成本低。
- 长程任务优势明确：horizon 敏感性曲线显示随轨迹变长优势扩大，正中长程 agentic 任务的痛点。
- 开源活跃：GitHub 仓库 (ZethWang/AgentOPSD) 已发布且持续更新，实验设置统一（同一 backbone/数据/预算对比全部基线），可控性好。

## 局限

- 依赖训练期特权 skill 检索（来自 SkillRL SkillBank），外部技能库的质量和覆盖度会直接影响证据信号质量，换领域需要另建技能库。
- 并非全面最优：WebShop 7B 的 Accuracy 上 SDAR（82.8）高于 AgentOPSD（79.7），优势主要体现在 Score 和 ALFWorld。
- 验证范围有限：只在 ALFWorld / WebShop / Search-QA 三个环境、Qwen2.5-3B/7B 两个尺度上评估，没有覆盖 SWE-bench 等主流编码智能体基准。

## 个人评价

这篇把"信用分配"从工程技巧提升到了贝叶斯推断的高度，思路和 3SPO、G2PO 这类改 advantage 构造的工作一脉相承但更简洁：核心洞察是"局部信号不是信用，信号对信念的修正才是"。对长程稀疏奖励的 agentic RL（包括我们关心的多步决策/路径规划类任务）很有参考价值——同样的"先验 + 证据累积 + 边际修正"框架不限于自蒸馏信号，任何回合级证据源（过程奖励、环境反馈）都能套。比较可惜的是实验环境偏传统（ALFWorld/WebShop），如果能上 SWE-bench 或 Terminal-Bench 这类更硬核的基准，说服力会强很多；另外 WebShop Acc 没赢 SDAR 也说明它在"分数优先"型环境更强，对"严格正确性"型任务的优势还需要更多证据。
