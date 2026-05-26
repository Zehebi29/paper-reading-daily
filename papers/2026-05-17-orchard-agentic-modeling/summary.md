# Orchard：微软开源Agent训练框架，30B模型SWE-bench 67.5%

**arXiv**: 2605.15040 | **GitHub**: https://github.com/microsoft/Orchard | **Microsoft Research**

---

## 一句话总结

微软开源了 Orchard，一个端到端 Agent 训练框架，含 K8s 原生环境服务和三种配方（SWE/GUI/Claw），在 SWE-bench Verified 上达到 67.5%（30B 模型——开源可比模型新 SOTA）。

## 问题背景

高性能 AI Agent 系统依赖专有代码库。开源框架聚焦于编排，缺乏可扩展的训练基础设施。

## 核心方法

### Orchard Env
三层架构：Client SDK → Orchestrator（FastAPI on K8s）→ In-Pod Agent。关键：Init Container Agent 注入、Pod-IP 直连。
成本：128 sandboxes × 240h → 按需 $3,362，spot $673（对比 E2B $7,078，Modal $10,305）。延迟：平均 0.28s。

### Orchard-SWE
Qwen3-30B-A3B-Thinking。SFT：64.3%。SFT+RL（BAR）：67.5%。关键：从失败轨迹中做信用分配 SFT。

### Orchard-GUI
Qwen3-VL-4B-Thinking。SFT：52.0% 平均。SFT+RL：68.4% 平均（WebVoyager 74.1%）。RL 提升 +16.4 分。

### Orchard-Claw
192 个合成任务，SFT+GRPO。Claw-Eval pass@3：59.6%。

## 亮点
1. 真正的开源训练框架（SFT+RL）
2. Orchard Env：Init Container 注入，成本仅为商业方案 1/10
3. 从失败中学习：信用分配 SFT
4. 跨领域：SWE/GUI/Claw 统一

## 局限
1. 30B 模型规模
2. Claw 仅 192 个任务
3. RL 训练复杂度
4. 依赖 K8s

## 个人评价
与 ToolCUA 形成对比：ToolCUA 聚焦 GUI RL pipeline，Orchard 聚焦基础设施。Init Container 注入非常巧妙——用 K8s 原生能力实现 Agent 训练环境。
