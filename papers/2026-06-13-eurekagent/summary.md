# EurekAgent: Agent Environment Engineering is All You Need For Autonomous Scientific Discovery

**arXiv**: 2606.13662 | **GitHub**: https://github.com/THU-Team-Eureka/EurekAgent | **机构**: Tsinghua University (THU)

---

## 一句话总结

自主科学发现的瓶颈不在于 design agent workflow，而在于 design agent environment；EurekAgent 通过四维度环境工程（权限/制品/预算/人在回路）协调现有 CLI Agent，在数学、kernel 工程、ML 工程三类任务上均取得新 SOTA，Circle Packing 任务总 API 成本不到 $11。

## 问题背景

现有 LLM Agent 做自主科学发现时，研究重心放在如何设计 agent 的工作流程（prompt、工具调用、反思链）。但随着模型能力提升，瓶颈转移到了环境层：agent 能做多好，取决于它所处的环境给了它什么资源、什么约束、什么反馈。这类比人类研究者——导师的作用不是替学生思考，而是提供好实验室、资源和监督。

## 核心方法

1. **权限工程 (Permissions Engineering)**: Docker 容器隔离 + 隐藏评分器（agent 无法篡改评分）+ GPU 助手 API 防资源争抢 + 同轮并行实现互不干扰
2. **制品工程 (Artifact Engineering)**: 文件系统 + Git 历史作为跨会话共享记忆，排名历史解方便快速复用强解
3. **预算工程 (Budget Engineering)**: 时间轴（截止时间提醒）+ 成本轴（跟踪 token 用量，对 agent 隐藏）+ 会话状态持久化支持中断恢复
4. **人在回路工程 (Human-in-the-loop)**: 终端 UI 显示各方案进度 + Web 监控可视化分数曲线，支持人类随时干预

## 关键结果

| 任务 | EurekAgent | 前方最佳 AI | 改善 |
|------|------------|------------------|------|
| Circle Packing (↑) | **2.635999** | 2.635986 | 新 SOTA |
| Erdős’ Min. Overlap (↓) | **0.380870** | 0.380876 | 新 SOTA |
| 1st Autocorr. Ineq. (↓) | **1.502861** | 1.502863 | 新 SOTA |
| TriMul Kernel (A100) | **2005.03 µs** | 2095 µs | +4.3% |
| MLE-Bench Lite Any Medal | **85.71%** | 71.43% | +14.3pt |
| MLE-Bench Lite Gold | **71.43%** | 57.14% | +14.3pt |

- Circle Packing 任务总 API 成本 < $11
- MLE-Bench 使用开源 LLM (GLM-5.1)，打败用商业模型的系统

## 亮点

1. **视角转换很刺激**: 从“design workflow”转向“design environment”，思路类似于软件工程中的“基础设施即代码”，把 agent 当“有能力但需要管理的实习生”来设计环境
2. **成本极低**: Circle Packing 新 SOTA 只花 $11，说明环境工程的 ROI 很高
3. **实用性强**: 四个维度都是可操作的工程实践，不是空洞的理论框架
4. **开源**: 代码和结果均已开源，可复现

## 局限

1. **依赖强 LLM 后端**: 系统协调的是 Claude Code 等 CLI Agent，对基座模型能力有隐含要求
2. **任务范围有限**: 实验只覆盖数学优化、kernel 工程、ML 工程三类，未覆盖更广泛的科学发现场景（如实验科学、理论推导）
3. **并行实现的同轮隔离机制未详细披露**: 多个实现 session 如何互不干扰的具体实现细节可以更透明

## 个人评价

这篇论文最大的启发是“环境工程”这个概念。过去我们老是想“怎么让 agent 更聪明”（提示词工程、工具设计、反思链），但这篇论文指出：当 agent 已经够聪明时，限制它的不是智力而是环境。这和现实世界中“好的研究环境比好的研究者更重要”的经验一致。对于做 Agent 系统的人来说，这提醒我们在优化 prompt 之余，也要花力气设计好 agent 的“实验室”——包括权限边界、反馈机制、资源管理、人类监督接口。
