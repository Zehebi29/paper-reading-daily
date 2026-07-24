# OpenForgeRL: Train Harness-native Agents in Any Environment

**arXiv**: 2607.21557 | **GitHub**: (will be released, CC BY 4.0) | **机构**: Microsoft Research + Columbia University + Dartmouth College

---

## 一句话总结

OpenForgeRL 是一个开源框架，通过轻量级代理 + Kubernetes 编排器，让任意 agent harness（如 Codex、OpenClaw、GUI agent）能在任意环境中端到端 RL 训练，在 ClawEval/OSWorld/WebVoyager 多个基准上超越同等规模开源模型。

## 问题背景

- 现代 AI agent 依赖复杂的推理 harness（如 Claude Code、Codex、OpenClaw）来管理多轮推理、工具调用和上下文，但这些 harness 的复杂状态逻辑使得端到端 RL 训练非常困难
- 现有的开源 RL 框架（如 veRL）无法原生表达有状态、多进程的 harness 推理，导致训练-部署不一致
- 研究者要么只能用简单的 ReAct 格式训练，要么完全跳过 RL，限制了 agent 能力的提升

## 核心方法

1. **轻量级代理（Lightweight Proxy）**：拦截 harness 的模型调用请求，一边转发给模型服务，一边将轨迹记录为标准 RL 训练数据，实现 harness 无关的数据收集
2. **Kubernetes 编排器**：每个 rollout 在独立的远程容器中运行，支持从 0 到数千个 rollout 容器的弹性伸缩，彻底解耦 rollout 计算和 GPU 训练
3. **任意 Harness x 任意环境**：框架支持 CLI agent（Codex/OpenClaw/ZeroClaw）、多模态 GUI agent（Molmo-Web/Qwen-Web）、浏览器/桌面操控 agent 等多种 harness
4. **标准 RL 后端**：无缝对接 veRL、slime 等开源 RL 代码库，使用 GRPO 等标准算法优化策略

## 关键结果

| 基准 | OpenForge-Claw (30B-A3B) SFT | SFT+RL | 对比基线 |
|:-----|:---:|:---:|:---:|
| ClawEval (pass@1) | 21.8 | 31.7 | - |
| ClawEval (pass@3) | 32.1 | 55.9 | - |
| QwenClawBench | 14.6 | 33.7 | - |
| MCPAtlas | 22.5 | 28.1 | - |

| 基准 | OpenForge-GUI (8B) SFT | SFT+RL | 对比基线 |
|:-----|:---:|:---:|:---:|
| OSWorld-Verified | 29.4 | 37.7 | 超过 UI-TARS-1.5-7B (28.2) |
| Online-Mind2Web | 38.7 | 63.0 | 超过 Qwen3-VL-8B (35.3) |
| WebVoyager | 49.2 | 72.3 | 超过 MolmoWeb-8B (78.2→49.2) |

- RL 带来持续且一致的提升：平均推理步数增加（更多探索），工具调用覆盖率提升
- 不同 harness 的学习难度差异显著：Codex 的 ReACT 格式最容易学，OpenClaw 的复杂工具定义更难
- RL 提升了自我验证（self-verification）和多步计划完成能力，但错误恢复仍是短板

## 亮点

1. **实用导向的框架设计**：轻量级 proxy + K8s 编排器的架构优雅地解决了 harness 训练的核心工程难题，真正实现了“任意 harness x 任意环境”
2. **全面的实验结果**：覆盖 6 种 harness、6 个环境、多个模型规模，展示了从 CLI agent 到 GUI agent 的广泛适用性
3. **深入的行为分析**：不仅报告结果，还系统分析了 RL 如何改变 agent 行为（工具覆盖率、自我验证、错误恢复等）
4. **开源友好**：CC BY 4.0 协议，代码、数据、模型都将开源，降低了复现门槛

## 局限

1. **代码尚未发布**：论文承诺将开源，但目前 repo 尚未公开，无法立即验证和复现
2. **结果依赖特定基础设施**：K8s 容器化 rollout 架构需要较强的 DevOps 支持，对资源有限的实验室不太友好
3. **错误恢复能力提升有限**：RL 虽然提升了工具覆盖和多步计划，但 error recovery 仍弱，说明稀疏奖励的局限性

## 个人评价

这篇论文是我最近看到的“最实用”的 agentic RL 工作之一。与那些提出新颖算法但难以落地的论文不同，OpenForgeRL 解决的是一个真实且痛苦的问题：训练带复杂 harness 的 agent。轻量级 proxy + K8s 编排的架构设计非常优雅，本质上是在 veRL 等训练框架和实际 agent harness 之间加了一个“翻译层”，让所有现有工具都能复用。对工业界部署 agentic RL 非常有参考价值——MSR 的团队确实懂工程痛点。
