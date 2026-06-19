# From Trainee to Trainer: LLM Designed Training Environment for RL with Multi-Agent Reasoning

**arXiv**: 2606.17682 | **GitHub**: https://github.com/LARK-AI-Lab/Trainee-to-Trainer | **HKUST(GZ) LARK Lab**

---

## 一句话总结

提出 LLM-as-Environment-Engineer 框架, 让当前 RL 策略模型自己分析失败轨迹并设计下一轮训练环境, 在 MAPF-FrozenLake 测试平台上, Qwen3-4B 超越 GPT-5.4、Grok-4.2、Gemini-3.1-Pro 等商业大模型.

## 问题背景

当前 RL 训练管线依赖人工手动重新配置训练环境, 需要专家经验来吨觉判断哪种配置能最好地提升当前策略. 这个手动过程效率低且难以扩展. 特别是在多智能体场景中, 环境配置的微调直接影响学习信号质量.

## 核心方法

- **LLM-as-Environment-Engineer**: 当前策略模型分析失败轨迹 + 上下文信息, 自动生成下一轮训练环境配置
- **MAPF-FrozenLake**: 新测试平台, 支持多维度环境配置 (data_ratio, hole_ratio, wait_ratio)
- **结构化上下文模块**: Failure Breakdown + Guideline + History + Summary + Training Details, 共五个信息模块指导环境设计
- **闭环自训练**: Train -> Validate -> Design -> Repeat, 每轮使用 GRPO 训练 + 策略模型作为环境工程师提出新配置
- **自诊断能力提升**: RL 检查点比原始基度模型更善于诊断自身弱点

## 关键结果

| 模型 | 3-agent Valid Rate | 3-agent Optimal Rate | 备注 |
|------|-----------|------------|------|
| Qwen3-4B + GRPO + Ours | 最高 | 最高 | 本方法 |
| Kimi-K2.5 | -5.20~-6.19 | -2.22~-3.43 | 最强商业 baseline |
| Qwen3-4B + GRPO (random) | -3.56~-11.25 | -1.89~-5.59 | 固定随机配置 |

- Qwen3-4B (4B参数) 超越 GPT-5.4、Grok-4.2、Gemini-3.1-Pro 等更大的商业模型
- 5个上下文模块的消融实验表明: Failure Breakdown 和 History 是最关键的两个信息源
- RL 检查点作为环境工程师优于原始基度模型

## 亮点

- 思路新颖: 让模型从被动学习者变成主动设计者, 形成自我迭代提升循环
- 实验设计严谨: 5个上下文模块的消融实验清晰展示了每个信息源的贡献
- 开源完整: GitHub 代码 + HuggingFace 数据集 + 模型权重, MIT 协议
- 实用价值: 4B 参数模型超越商业大模型, 证明了环境工程师策略的有效性

## 局限

- MAPF-FrozenLake 是网格场景, 与真实世界的复杂场景有差距, 泛化性有待验证
- 训练过程需要多轮迭代, 计算成本高于单次训练
- 环境工程师的输出质量依赖于失败轨迹的质量, 初始配置随机可能导致早期迭代效率低

## 个人评价

这篇论文的核心洞察——训练好的模型比未训练的模型更擅长诊断自己的弱点——很符合直觉但之前没有被系统地实验证过. 这个思路与 EurekAgent 的环境工程、SkillAxe 的自我迭代优化有着相似的哲学: 让模型自己来改进自己的学习过程. 不过当前的测试场景还比较简单 (FrozenLake 网格), 能否扩展到更复杂的任务 (如 WebShop、SWE-bench) 是关键的下一步. 开源生态做得很好, 代码、数据集、模型权重都可用.
