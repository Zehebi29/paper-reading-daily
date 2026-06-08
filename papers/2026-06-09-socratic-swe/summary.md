# Socratic-SWE: Self-Evolving Coding Agents via Trace-Derived Agent Skills

**arXiv**: 2606.07412 | **GitHub**: 无 | **机构**: Alibaba Group & Shanghai Jiao Tong University

---

## 一句话总结

Socratic-SWE 通过"trace -> skill -> task -> trace"闭环自进化框架, 让 SWE agent 从自己的历史求解轨迹中提取技能并生成新训练任务, 在 SWE-bench Verified 上达到 50.40% (比 base agent +7.80).

## 问题背景

现有 SWE agent 训练依赖静态数据合成, 无法随 agent 能力进化调整训练分布. agent 自身的失败轨迹包含丰富诊断信号, 但通常被丢弃. 核心问题: 如何让 agent 从自身经验中持续学习?

## 核心方法

1. **Agent Skill Registry**: 从历史成功/失败轨迹中蒸馏出结构化技能 (名称 + 适用条件 + 操作序列), 按语义去重并按轨迹覆盖率过滤
2. **Skill-Guided Task Generator**: 给定仓库和一个采样技能, 生成候选修复任务; 用梯度对齐奖励 `R_G = Valid * cos(g_tau, G_v)` 筛选任务, 确保训练方向与验证集改进方向一致
3. **Repository Repair Solver**: agent 在仓库沙箱中生成 patch, 用可执行反馈奖励 (全量通过 + 部分修复率 + 回归避免)
4. **Joint Optimization**: Generator 和 Solver 共享同一 policy, 分别用 GRPO 和 GDPO (Group Reward-Decoupled Policy Optimization) 联合训练

## 关键结果

| Benchmark | Base Agent | Socratic-SWE | Gain |
|:---|:---|:---|:---|
| SWE-bench Verified | 42.60% | 50.40% | +7.80 |
| SWE-bench Lite | 29.67% | 36.67% | +7.00 |
| SWE-bench Pro | 17.24% | 22.85% | +5.61 |
| Terminal-Bench 2.0 | 10.11% | 14.61% | +4.50 |

- 5 轮迭代可达 52.00% (Verified), 高于最强 baseline SSR 的 48.00%
- 在固定 36k 训练实例预算下 (3 轮 x 12k), 超越所有 baseline (R-Zero, SPIRAL, Absolute-Zero, Socratic-Zero, SSR)
- Skill Registry 是最关键组件: 移除后 Verified 降 4.20 点

## 亮点

- 闭环自进化设计, 训练数据随 agent 能力自动调整, 无需人工构造
- 梯度对齐奖励机制确保生成的训练任务真正有效 (比 hardness-based 和 variance-based 选择高 3 点)
- GDPO 处理多组件奖励 (全量通过 + 部分修复 + 回归避免), 比标准 GRPO 高 1.8 点
- 跨任务泛化: 在 SWE 上训练的技能可迁移到 Terminal-Bench

## 局限

- 依赖强 base model 作为 skill extractor 和 solver, 弱模型效果未验证
- 36k 训练实例 x 3 轮的计算成本较高
- 仅在 Python 仓库上验证, 跨语言泛化未知
- 梯度对齐奖励需要额外前向传播计算梯度方向, 增加了 generator 的计算开销

## 个人评价

这篇工作最打动我的是"梯度对齐奖励"这个设计 -- 不是简单按难度或方差选任务, 而是直接衡量候选任务的梯度方向是否与验证集一致. 这比 curriculum learning 的常见启发式 (如按 loss 排序) 更有理论依据. 对我们做 agentic RL 的人来说, 这个思路可以直接借鉴: 在 agent RL 中, 哪些 rollout 轨迹值得用来训练, 也可以用类似的梯度对齐来筛选.

另一个值得注意的点是 Skill Registry 的设计. 从轨迹中蒸馏技能 -> 去重 -> 按覆盖率过滤, 这个 pipeline 本身就是一个通用的 agent experience compression 方法, 和 ReuseRL (2605.31509) 的 MDL-based skill reuse 有异曲同工之处.
