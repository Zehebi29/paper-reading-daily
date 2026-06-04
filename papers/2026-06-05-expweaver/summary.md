# ExpWeaver: LLM Agents Learn from Experience via Latent RAG

arXiv: 2606.01041 | GitHub: ulab-uiuc/ExpWeaver | ICML 2026

## 一句话总结
ExpWeaver 让 LLM Agent 直接在隐空间中检索和融合历史经验, 无需独立的 RAG 模块, 无需拼接文本到上下文窗口. 端到端 GRPO 训练, 13 个任务里 SOTA 了 12 个, 且 token 开销仅比无检索方法多约 10%.

## 问题背景

当前 LLM Agent 通过检索历史经验来提升推理和决策能力已越来越普遍, 但现有方法全是文本级操作: 用单独的检索器根据语义相似度选取文本片段, 拼接进上下文窗口让 LLM 读. 两个核心问题: (1) token 开销巨大, 上下文窗口膨胀 1.5-2 倍; (2) 检索器和生成器分离架构, 无法端到端联合优化.

## 核心方法

1. 经验隐空间编码: 每条交互经验(query, reasoning trace, output, reward)先被 LLM 自己压缩成文本摘要, 再用同一个 LLM 的最后一层 hidden state 编码为隐向量存入经验库, 保证在策略对齐.
2. 逐步隐空间检索: 每个 decoding step, 当前 hidden state 作为 query, 在经验库中用余弦相似度检索 Top-K 经验向量, 完全不需要独立检索器.
3. 交叉注意力融合: 用可学习的 query token 对检索到的经验向量做 cross-attention 聚合, 再按 norm 对齐缩放.
4. 门控残差整合: 通过 retention gate, input gate 和 mixing coefficient, 用 norm-preserving interpolation 注入经验信息, 保留原始推理能力同时融入经验.
5. 端到端 GRPO 训练: 所有新增参数(query token, gate 矩阵, lambda)与 LLM 参数一起用 GRPO 联合优化. Qwen2.5-3B-Instruct + LoRA(rank=32), 4xA6000 即可训练.

## 关键结果

| 基准 | ExpWeaver | 最强基线 | 提升 |
|------|-----------|----------|------|
| ExpBench-Generic(10任务平均) | 78.25% | 73.27% (Search-R1) | +4.98% |
| ExpBench-Sci(Chem-TDC) | 69.58% | 63.78% (Search-R1) | +5.80% |
| ExpBench-Rec(NDCG@10 平均) | 44.40% | 37.90% (IRanker) | +6.50% |
| 跨域 zero-shot 迁移 | +16.32% | vs 最强基线 | -- |
| 跨域 few-shot 迁移 | +15.21% | vs 最强基线 | -- |

- 13 个任务中 12 个达到 SOTA
- token 开销仅比无检索方法多约 10%, Search-R1 等文本检索方法需要 1.5-2x tokens
- 文本级检索方法(ReasoningBank 等)通用任务平均仅 60-63%, 被大幅甩开

## 亮点

1. 范式创新: 首次将 Agent 经验学习从拼接文本->LLM 阅读升维为隐空间检索->门控融合, 消除了分离式 RAG 架构的所有弊端.
2. 高性价比: 3B 模型 + LoRA + 4xA6000 即可训练, 性能超过多款 7B+ 模型方案, token 开销极低.
3. 通用性: 同时支持生成式任务(QA, 推理, 编程, 科学)和排序任务(推荐), 一个框架打全场.
4. 跨域泛化惊人: zero-shot +16.32%, few-shot +15.21%, 说明隐空间经验表征捕捉到了真正的可迁移知识模式.

## 局限

1. 只在 3B 模型上做了实验, 隐空间 RAG 能否扩展到 7B/13B/更大模型尚待验证 -- 模型越大 hidden state 维度越高, 检索效率和门控机制的表现需要重新审视.
2. 经验库使用简单 FIFO 淘汰策略, 缺乏对经验质量, 时效性, 多样性的精细管理; 在长期持续学习场景中可能被噪声污染.
3. 论文没有讨论多轮交互式 Agent 场景(如 WebArena, SWE-bench), 当前实验以单轮生成/排序为主, 离真实 Agent 闭环交互还有距离.

## 个人评价

ExpWeaver 是我近期读过最干净的 Agent 经验学习方案. 它在方法上做了一个很优雅的减法: 把 RAG 的检索器, 文本拼接, 上下文窗口管理全部砍掉, 换成一个更本质的隐空间记忆+门控调用机制. 这让我想到人类记忆 -- 我们回忆经验时并不是把整段文字复制到工作记忆里, 而是激活一个感觉来影响当前决策. ExpWeaver 的 gated residual integration 正是这个思路的工程化实现.

对 GWS 研究的启发: 我们的焊缝路径规划 Agent 也有大量可复用的操作经验(哪种焊接策略在什么几何条件下效果好), 但目前还停留在简单的 prompt 模板. 如果能在 Agent 的 RL 训练中引入类似 ExpWeaver 的隐空间经验机制 -- 每个成功的焊缝规划 rollout 编码为隐向量, 在下一次遇到相似几何结构时自动激活相关经验 -- 或许能显著提升长尾场景的泛化能力. LoRA + 小模型的方案也让这个想法在工程上可行.
