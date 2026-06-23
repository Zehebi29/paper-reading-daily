# Tmax: A Simple Recipe for Terminal Agents

**arXiv**: 2606.23321 | **GitHub**: https://github.com/hamishivi/tmax | **机构**: Allen Institute for AI, University of Washington

---

## 一句话总结

TMAX 提出了一个简洁的开源 RL 训练方案, 仅用 9B 参数就在 Terminal-Bench 2.0 上达到 27.2% 准确率, 超过许多 30B+ 的闭源模型, 同时公开了数据/模型/代码全套资源.

## 问题背景

终端 (terminal) 是 AI agent 最重要的执行环境之一, 但开源社区在终端 agent 训练上面临两个瓶颈: (1) 缺乏大规模、多样化的 RL 训练环境数据; (2) 没有经过验证的、可复现的 RL 训练 recipe. 现有开源数据集 (如 Endless Terminals 仅 2,400 条) 规模太小, 难以支撑长 horizon 的 agent RL 训练.

## 核心方法

- **TMAX-15K 数据集**: 14,600 个合成 RL 环境实例, 比之前最大的开源终端数据集大 2.5 倍. 使用 Gemini-3-Pro 生成, 通过 9 个结构化轴 (Domain, Skills, Task Complexity, Verifier Kind 等) 实现组合多样性.
- **组合式难度控制**: 双复杂度轴 (Task + Command complexity) + 渐进式验证器 (metric-threshold, fuzz-equivalence) 避免 bimodal 难度分布.
- **Soft Filtering**: 跳过昂贵的 teacher 验证, 依赖 RL 训练自然过滤 pass rate 为 0% 的环境, 大幅降低数据生成成本.
- **DPPO 算法**: 使用 Divergence Proximal Policy Optimization 替代 GRPO, 在长 horizon agent RL 中更稳定.
- **FP32 LM Head 稳定化**: 关键技术细节 -- 对语言模型 head 使用 FP32 精度, 显著减少 Qwen 3.5 等混合模型的训练-推理 logprob 不匹配.

## 关键结果

| 模型 | 参数量 | Terminal-Bench 2.0 |
|:---|:---:|:---:|
| TMAX-4B | 4B | 18.9% |
| TMAX-9B | 9B | **27.2%** |
| TMAX-27B | 27B | **42.7%** |
| Qwen 3.5 9B (baseline) | 9B | 21.1% |
| Endless-8B | 8B | ~10% |
| Claude Haiku 4.5 | - | ~27% |

- TMAX-9B 以 9B 参数追平 Claude Haiku 4.5, 超过所有 <30B 开源模型
- 跨任务泛化: SWE-Bench Verified +9.5%, AIME +23.6% (单轮)
- 跨 harness 泛化: 在 OpenHands, Terminus-2 等不同评估框架下增益一致 (最少 +9pt)
- 跨模型族: 配方同样提升了 Qwen 3 8B

## 亮点

- 全套开源: 数据 (HuggingFace: allenai/TMax-15K), 模型, 代码 (GitHub), CC BY 4.0
- 简洁但有效: 仅用 outcome-only RL (无 PRM/process reward), 就达到了很强的效果
- FP32 LM Head 是一个容易被忽视但影响很大的工程细节, 对混合架构模型 (如 Qwen 3.5) 尤其关键
- 数据生成的组合式设计 (9 轴) 很有工程品味, 比随机生成更高效

## 局限

- Terminal-Bench 2.0 绝对分数仍然不高 (27% @ 9B), 说明终端 agent 还远未解决
- 合成数据依赖 Gemini-3-Pro 生成, 数据质量上限受限于教师模型能力
- 论文未详细讨论 RL 训练的计算成本 (仅提到 一行 RL step 约 $1K), 门槛不透明

## 个人评价

这篇论文的核心价值不在于方法创新, 而在于提供了一个经过充分验证的开源 baseline recipe. 对于想做 terminal agent RL 的研究者来说, 这是一个可以直接上手的起点. FP32 LM Head 这个工程细节特别值得注意 -- 它说明在 agent RL 训练中, 数值稳定性问题比算法选择更常被忽视但影响更大.

从 agent RL 的宏观视角看, TMAX 进一步验证了一个趋势: outcome-only RL (不需要 process reward model) 在 agentic 场景下已经足够强, 只要数据够大够多样. 这和之前 PaW, 3SPO 等工作的发现一致.
