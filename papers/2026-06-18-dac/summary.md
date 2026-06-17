# DAC: Divide and Cooperate - Role-Decomposed Multi-Agent LLM Training with Cross-Agent Learning Signals

**arXiv**: 2606.10684 | **GitHub**: None | **机构**: Seoul National University (SNU), SKI-ML Lab

---

## 一句话总结

DAC 把 agentic search 分解成 Searcher + Generator 两个角色, 用 cross-agent reward 信号做 credit assignment, 仅用 LoRA (2.12% 参数) 就超过了全量微调的 Search-R1.

## 问题背景

现有 agentic search 方法把 evidence acquisition 和 answer generation 塞进同一个 policy, 导致两个问题: (1) policy space 组合爆炸, 探索效率低; (2) credit assignment 困难 - 搜索到了足够证据但生成错了, 搜索端也会被惩罚, 反之亦然. 这在 multi-hop QA 尤其严重.

## 核心方法

1. **角色分解**: 搜索任务拆成 Searcher S (迭代检索证据) 和 Generator G (基于证据生成答案) 两个独立 agent
2. **跨 agent 奖励设计**: Searcher 奖励条件是"证据充分 AND Generator 没 abstain"; Generator 奖励条件是"答对 OR 证据不足时正确 abstain"
3. **Hard-positive augmentation**: 给充分证据集里混入无关噪声文档, 防止 Generator 变得过度保守 (总 abstain)
4. **Turn-level difference rewards**: 每轮搜索给 Searcher 密集反馈 - 本轮检索后下游奖励的变化量
5. **参数高效实现**: Searcher 和 Generator 各自挂一个 LoRA adapter (rank 32), 共享 backbone, 总参数量仅 2.12%

## 关键结果

| 方法 | 微调方式 | Avg EM | Avg LLM Score |
|:---|:---|:---:|:---:|
| Search-R1 (best) | Full FT | 39.6 | 49.0 |
| **DAC (ours)** | **LoRA** | **39.8** | **50.7** |

- 在 Qwen2.5-7B-Instruct 和 Qwen3-8B 两个 backbone 上均一致优于 baselines
- 7 个 QA benchmark (NQ, TriviaQA, PopQA, HotpotQA, 2WikiMultihopQA, MusiQue, Bamboogle)
- Generator 的 abstain accuracy 和 selective accuracy 均显著高于 Search-R1

## 亮点

- Cross-agent reward 设计很巧妙: Generator 的 abstention 天然成为 Searcher 的 reward signal, 解耦了 credit assignment
- Hard-positive augmentation 有 Stackelberg game 理论支撑, 不是 heuristic
- LoRA 就够了 - 2.12% 参数量超过全量微调, 实用价值高
- Abstention 机制可以独立部署, 比如证据不足时触发额外搜索轮次

## 局限

- 仅在 QA 任务上验证, 没有扩展到更广泛的 agent 任务 (如 web browsing, code generation)
- Generator 的 abstention 判定依赖 EM-based 的外部验证, 不一定总能获得
- 没有公开代码, 可复现性待确认
- 没有和最新的 agent RL 方法 (如 3SPO, ECPO) 做对比

## 个人评价

这篇论文的核心 insight 非常清晰: agentic search 本质上是两个不同能力的协作, 硬塞在一个 policy 里是浪费. 跨 agent reward 的设计 (尤其是 abstention 作为 verification signal) 是一个可以推广到其他 multi-agent 协作场景的 pattern. Hard-positive augmentation 的 Stackelberg 分析也很扎实, 比纯 empirical 的 work 好不少.

不过最让我感兴趣的是 LoRA 就能超过全量微调这个结果 - 这说明角色分解本身带来的 inductive bias 比增加参数更重要. 这对 agent RL 的 scaling 方向有启发: 不一定要训更大的模型, 把任务结构拆对可能更有效.
