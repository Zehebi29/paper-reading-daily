# Scaling the Horizon, Not the Parameters: Reaching Trillion-Parameter Performance with a 35B Agent

**arXiv**: 2606.30616 | **GitHub**: https://github.com/InternScience/Agents-A1 | **机构**: 上海人工智能实验室 (Shanghai AI Lab) / InternScience

---

## 一句话总结

Agents-A1 是一个 35B MoE 智能体模型，通过扩展智能体视界（agent horizon scaling）而非参数规模，在长视界任务上达到甚至超越万亿参数模型（如 Kimi-K2.6、DeepSeek-V4-pro）的性能。

## 问题背景

当前 LLM 智能体模型面临一个根本矛盾：长视界任务（long-horizon agent tasks）需要模型在数千步的推理、搜索和工具调用中保持连贯，但传统的参数规模扩展路线（scaling law）在 agent 场景下遭遇瓶颈。单纯把模型做到万亿参数级在部署成本和推理效率上都不可持续。问题是：能否在不增加模型参数的前提下，通过扩展智能体的“视界”来提升能力？

## 核心方法

1. **长视界知识-动作基础设施**：构建了一套将外部知识、动作、观察结果和验证器输出连接起来的基础设施，生成平均长达 45K token 的智能体轨迹数据。

2. **三阶段训练方案**：第一阶段对所有领域进行 SFT 对齐基础模型；第二阶段训练领域级教师模型（domain-level teacher），捕获每个领域专长；第三阶段提出多教师域路由在线蒸馏（multi-teacher domain-routed on-policy distillation），通过显著词汇对齐（salient vocabulary alignment）提升跨领域知识迁移效率。

3. **六域统一**：将搜索、工程、科学研究、指令遵循、通用智能体任务和科学智能体任务六个异质领域统一到一个可部署的学生模型中。

4. **RL 后训练增强**：在蒸馏后对教师模型进行长上下文 RL 训练，进一步强化检索和指令遵循能力。

## 关键结果

| Benchmark | 类型 | Agents-A1 (35B) | 最佳同行 (35B) | 万亿参数 SOTA (1T) |
|:---|:---|:---|:---|:---|
| SEAL-0 | 长视界搜索 | **56.4** | 49.6 (Nex-N2-mini) | 55.0 (DeepSeek-V4-pro) |
| IFBench | 指令遵循 | **80.6** | 70.2 (Qwen3.5-35B) | 75.9 (GPT-5.5) |
| HiPhO | 科学研究 | **46.4** | 38.5 (Nex-N2-mini) | 43.3 (GPT-5.5) |
| FrontierScience-Olympiad | 科学研究 | **79.0** | 64.5 (Qwen3.5-35B) | 78.0 (GPT-5.5) |
| FrontierScience-Research | 科学研究 | **40.0** | 5.0 (Nex-N2-mini) | 26.7 (GPT-5.5) |
| BrowseComp | 长视界搜索 | 75.5 | 74.1 (Nex-N2-mini) | 84.4 (GPT-5.5) |
| GAIA | 通用智能体 | 96.0 | 82.5 (Nex-N2-mini) | 98.1 (DeepSeek-V4-pro) |
| IFEval | 指令遵循 | **94.8** | 91.9 (Qwen3.5-35B) | 93.5 (GPT-5.5) |
| MolBench-bind | 科学智能体 | 56.8 | 51.4 (Nex-N2-mini) | 62.2 (GPT-5.5) |

## 亮点

- **horizon scaling 新范式**：明确提出“扩展视界而非参数”的智能体模型 scaling 路线，在 35B 量级达到万亿参数模型水平，部署成本大幅降低
- **多教师域路由蒸馏**：domain-routed on-policy distillation + salient vocabulary alignment 是一种新颖且实用的知识迁移方案，解决了多领域专长聚合问题
- **45K token 长轨迹基础设施**：构建了远超常规数据规模的长视界 agent 轨迹数据管道，为社区提供了可复用的资源
- **全面开源**：模型 checkpoint（HuggingFace + ModelScope）、评估框架、代码全部开源，Apache-2.0 许可

## 局限

- **科学 agent 任务仍有差距**：在 MatTools (47.1 vs 68.8) 和 MLE-Lite (43.9 vs 72.7) 上与 GPT-5.5 等顶尖模型差距较大，说明纯蒸馏路线在高度专业化的科学任务上仍有天花板
- **仅报告了对比结果**：论文属于技术报告（tech report），缺少对 horizon scaling 本身规律的深入分析（如 scaling curve、数据量-性能关系等）
- **45K 轨迹数据未开源**：虽然模型开源，但核心的长视界训练数据是否发布未明确说明

## 个人评价

这篇论文提供了一个非常务实的视角：与其追逐万亿参数模型（部署即地狱），不如在中等规模（35B）上通过更好的数据和训练策略来达到同等性能。domain-routed on-policy distillation 这个技术点对我们自己的 GWS 项目也有启发 —— 把不同焊缝类型（对接/角接/搭接）视为不同“domain”，各自训练专门路径规划策略再蒸馏到统一模型，可能是一个值得尝试的方向。另外，45K token 的 agent 轨迹长度也说明，长上下文支持（262K context）是当前 agent 模型不可或缺的基础能力。

这篇是 InternScience（上海 AI Lab）的作品，第一作者 Lei Bai 是上海 AI Lab 的青年研究员，论文有 50+ 作者，显露出大规模团队作战的组织能力。
