# Pass the Baton: Trajectory-Relayed On-Policy Distillation

**arXiv**: 2607.26057 | **GitHub**: https://github.com/ZJU-REAL/Relay-OPD | **机构**: Zhejiang University + Alibaba Group

---

## 一句话总结

Relay-OPD 提出了一种“接力赛”式的 on-policy distillation 方法，当检测到学生模型走错方向时，老师模型短暂接管生成，纠正后再将“棒”传回给学生，在 8 个数学推理 benchmark 上平均超过标准 OPD +5.73%，同时减少训练轨迹长度超过 50%。

## 问题背景

On-policy distillation (OPD) 是当前对 LLM 进行知识蒸馏的核心技术，但它存在一个根本性问题：“前缀失败”（prefix failure）。一旦学生在早期推理步骤中走了错误方向，后续所有生成都会基于这个错误建立，导致训练资源大量浪费。主观上去判断“什么时候学生走错了”需要额外的 reward model 或 verifier，成本很高。

## 核心方法

1. **Handoff Trigger — 无标签检测前缀失败**: 观察到当学生走错方向时，老师模型倾向于用“But”“Wait”等反思标记纠正，而学生继续沿原方向生成。这个差异可以作为 label-free 的接力触发器。

2. **Relay Trajectory**: 触发后老师短暂接管生成 L 段落（teacher leg）纠正推理方向，然后学生恢复生成并在得到的 relay trajectory 上优化。

3. **Relay Budget (M, L)**: 限制老师接力的次数 M 和每次接力的长度 L，确保干预早期、局部、不过度偏离学生策略。

4. **Speculative Decoding 架构**: 整个 rollout 在单个 speculative decoding 引擎中执行（学生 draft，老师 verify），无额外延迟。

## 关键结果

| 实验 | 结果 |
|:---|:---|
| 学生: Qwen3-1.7B-Non-Thinking, 8 benchmark avg | Relay-OPD +5.73% vs standard OPD, +1.49% vs FastOPD |
| 学生: Qwen3-0.6B-Non-Thinking | 一致性提升，所有 benchmark 最优或次优 |
| 训练轨迹长度 | 减少超过 50% |
| 仅替换反思 token (L=0) | Teacher tokens 仅占 0.35%，准确率从 27.73 提升至 34.96 (+7.23%) |
| L=3 + 延迟触发 | 延迟一步准确率从 41.99 降至 33.98 |

## 亮点

1. **无需额外标注或 reward model**: handoff trigger 是完全 label-free 的，仅依赖 teacher/student 的继续偏好差异
2. **大幅减少训练成本**: 轨迹长度减半意味着同样的 GPU 可以训练更多 step，或节省资源
3. **原理清晰且通用**: 只要 teacher 比 student 强，这个 asymmetry 就存在，应该能拓展到 agent 训练场景
4. **开源代码**: GitHub 已公开，基于 speculative decoding 架构，实现精美

## 局限

1. 实验仅在数学推理场景验证，尚未在工具调用、多轮交互等 agentic 任务上测试
2. Teacher 的纠正可能介绍学生模型未见过的推理模式，影响 student 自身探索能力
3. Handoff trigger 的设计依赖于 reflection token 的存在，对于不使用这类 token 的模型可能无效

## 个人评价

这篇工作的核心观察很有意思——老师和学生在错误路径上的继续行为差异可以作为无标签的“走错了”信号。对我们 GWS 研究而言，这种“接力赛”思想在 agent 训练中同样有很大潜力——当 agent 在多步骤任务中走偏时，不必重头开始，而是让强大模型“接棒”一下再传回来。特别是在焊缝路径规划中，某些步骤的纠错明显比重头规划更经济，Relay-OPD 的思路很值得借鉴。
