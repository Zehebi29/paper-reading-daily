# Agon: Competitive Cross-Model RL with Implicit Rival Grading of Reasoning

**arXiv**: 2607.07690 | **GitHub**: 无 | **作者**: Vladislav Beliaev

---

## 一句话总结

Agon 让两个网络互为对手的评级器——一方撰稿、另一方批阅同时解题，以“能否超越知己知彼的对手”作为奖励，在 Qwen3 上 DeepMath 硬分割任务中将 GRPO 的 pass@1 翻了一倍。

## 问题背景

当前 RL 从可验证奖励（如 GRPO）只评分最终答案，不评分推理过程。对于难题，这使得模型只会写更多而非思考更好——因为推理过程本身从未被评分，也没有“良好思考”的标签。Process Reward Model (PRM) 需要大量人工标注，成本高昂且难以扩展。

## 核心方法

1. **对抗性培训架构**: 两个网络轮流扮演 draft 和 challenge 角色，对彼此的推理过程进行暗示性评分
2. **Implicit Rival Grading**: draft 方写解法，challenge 方批阅并同时自己解，每方因“能否干掉对手”获得奖励，不需要人工标注
3. **Co-evolution**: 双方同时优化，面临不断增强的对手，达到单模型 RL 无法提供的持续进步
4. **推理阶段的 cascade**: 部署时也保持双网络结构，draft 写完 challenge 审查并回答

## 关键结果

- **DeepMath 硬分割**: pass@1 翻倍（vs GRPO），约为未训练 MoA baseline 的 8x
- **竞争编程**: 在 competitive programming 上复制了 DeepMath 的提升规律
- **模型泛化**: 在 Qwen3.5 和 Gemma 4 上均复现了显著提升
- 不需要人工标注的过程标签，不需要 reward model，只需两个实力相当、行为差异化的网络

## 亮点

1. 很新颖的设计——用对抗性学习替代人工标注的 PRM，解决了推理过程评分的核心矛盾
2. 效果显著：pass@1 翻倍，且在多个模型家族和任务类型上一致
3. 方法简洁，不需要额外的数据标注或 reward model 训练
4. 对 LLM Agent 多代理场景有启发——多个 agent 之间可以互相评分彼此的行为

## 局限

1. 未提供开源代码，复现难度大
2. 需要两个实力相当的网络，训练成本翻倍
3. 主要在数学推理和竞争编程上评测，未覆盖更广泛的代理场景

## 个人评价

Agon 是一篇思路很清奇的短文章，它把“不好评的推理过程”转化为“两个模型互相评比”的对抗问题，设计很巧妙。对我们的启示是：多代理场景下，同一任务的多个 agent 可以相互作为对方的 grader，这可能比单独的 reward model 更有效。但未开源代码和双倍训练成本是主要瘦点。
