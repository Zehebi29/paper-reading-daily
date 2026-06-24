# OpenThoughts-Agent: Data Recipes for Agentic Models

**arXiv**: 2606.24855 | **GitHub**: https://github.com/open-thoughts/OpenThoughts-Agent | **机构**: Stanford, UC Berkeley, UT Austin, NYU, UW, UCSD, ASU, CMU, UCLA, UNC Chapel Hill, TUM, LAION (50+ researchers)

---

## 一句话总结

OpenThoughts-Agent (OT-Agent) 通过 100+ 控制变量实验系统性地研究了 agentic model 的数据配方, 用 100K 条精选数据微调 Qwen3-32B 在 7 个 agentic benchmark 上平均达到 44.8%, 比此前最强开源数据模型 Nemotron-Terminal-32B (40.9%) 高出 3.9 个百分点.

## 问题背景

Agentic language model 的应用前景广阔, 但公开社区对如何为通用 agent 模型策划训练数据几乎一无所知. 现有的开源努力 (SWE-Smith, SERA, Nemotron-Terminal, OpenSWE) 通常只针对单一 benchmark (SWE-Bench 或 Terminal-Bench), 导致训练出的模型缺乏跨任务泛化能力. OT-Agent 项目旨在填补这一空白, 构建一个完全开放的端到端数据策划流水线.

## 核心方法

- **系统化数据策划流水线**: 通过 100+ 控制变量实验, 系统研究流水线每个阶段 (任务选择, 轨迹生成, 数据过滤) 的最优配置
- **任务来源多样化**: 混合 Top-4 任务生成策略 (swe-smith, stackexchange-superuser, stackexchange-tezos, issue-tasks) 优于使用单一最佳来源, 提升跨 benchmark 泛化
- **长轨迹过滤**: 保畗 >=5 个 model turn 的 agent rollout 作为质量信号, 显著提升训练效果
- **合成任务扩增**: 对稀缺来源进行 surface form 扩增 (Tezos 子集从 ~900 扩到 21K), 配合性能加权采样实现 scaling
- **RL 数据源优化**: 发现 pymethods2test (将竞赛题重构为 Python contract) 作为 RL 数据源效果最佳, 诱导出 compact explore-patch-submit 策略

## 关键结果

| Benchmark | OT-Agent (32B) | Nemotron-Terminal-32B | 提升 |
|:----------|:--------------:|:---------------------:|:----:|
| 7 Benchmark 平均 | 44.8% | 40.9% | +3.9pp |
| SWE-Bench Verified | 54.0% | 41.9% | +12.1pp |
| Terminal-Bench 2.0 | 26.2% | 25.1% | +1.1pp |

**Ablation 关键发现:**

| 发现 | 影响 |
|:-----|:-----|
| 任务来源选择 | SWE-Bench 上可造成 ~30pp 差异 |
| 最佳模型 != 最佳教师 | GPT-5.3-Codex 做教师反而比 GLM-4.7-AWQ 差 ~5% |
| 长轨迹更好 | >=5 turns 的 rollout 显著优于短轨迹 |
| 任务扩增无效 | LLM 对任务描述的 refine/augment 不提升效果 |
| RL 数据源 | 不同源导致 7.6pp 的平均准确率差异 |
| SFT + RL | 8B 模型上 SFT+RL 组合优于纯 SFT 和纯 RL |

## 亮点

- 完全开源: 训练集, 数据流水线, 实验数据, 模型全部公开 (openthoughts.ai)
- 100+ 控制变量实验提供了迄今最系统的 agentic training data 研究
- 发现 最佳模型 != 最佳教师 等反直觉结论, 对社区有实际指导价值
- 展示了强 scaling 性质: 在每个训练集规模下都优于替代开源数据集

## 局限

- 评估集中在 terminal/SWE 类 benchmark, 对其他 agent 范式 (web browsing, tool orchestration) 的泛化性未充分验证
- 32B 模型仍需大量计算资源, 8B 模型的性能提升相对有限
- 数据策划流水线依赖多个 proprietary teacher model (GPT-5.3-Codex, GLM-4.7-AWQ), 完全复现仍有门槛

## 个人评价

这篇论文最有价值的贡献不是最终模型的分数, 而是那 100+ ablation 实验揭示的数据策划规律. 最佳模型 != 最佳教师 和 任务扩增无效 这两个发现尤其值得注意 -- 它们直接挑战了社区的常见假设. 对于做 agentic RL 的人来说, RL 数据源选择导致 7.6pp 差异这个发现也很有启发: 不是所有带 reward signal 的数据都一样好, 数据的 形状 (诱导出的 policy pattern) 可能比数据量更重要. 项目团队规模 (50+ 人, 顶校联合) 说明这类系统性数据研究确实需要大规模协作, 单打独斗很难做到这个深度.
