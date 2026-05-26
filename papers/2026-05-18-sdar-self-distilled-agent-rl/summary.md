# SDAR：自蒸馏门控Agent强化学习

> arXiv: 2605.15155v1 | 2026-05-14 | cs.LG / cs.AI / cs.CL
> 代码: https://github.com/ZJU-REAL/SDAR

## 问题

经过 RL 训练的 LLM Agent 在多轮任务中面临稀疏的轨迹级奖励问题。On-Policy Self-Distillation (OPSD) 通过具有特权上下文的教师模型提供 token 级密集监督，但简单地将 GRPO + OPSD 组合起来会导致多轮 Agent 灾难性失败。

## 方法：SDAR = GRPO + 门控 OPSD

1. **RL 主干**：GRPO 作为主要优化目标
2. **OPSD 辅助**：具有特权上下文的教师模型计算 token 级 log-probability 差距
3. **Sigmoid 门控（核心）**：自适应门控按 token 调节蒸馏强度。正差距信任、负差距衰减。门控从梯度流中分离。
4. **技能检索**：UCB bandit 或从 SkillBank 关键词匹配

## 结果

| Benchmark | vs GRPO |
|:---|---:|
| ALFWorld | +9.4% |
| Search-QA | +7.0% |
| WebShop-Acc | +10.2% |
| WebShop-Reward | +5.2 |

模型：Qwen2.5-3B/7B, Qwen3-1.7B。开源：github.com/ZJU-REAL/SDAR

## 优势

- 优雅：每个 token 自己决定自己的监督强度
- 非对称信任：正差距 = 认可；负差距不一定代表拒绝
- 多模型验证，代码可用
- 门控稳定了 RL 训练

## 局限

- 需要 SkillBank 和检索模块
- 依赖于任务的特权上下文设计
- 仅限文本环境，无视觉/工具使用设置
- 仅小模型（1.7B-7B），大模型扩展性未知

## 个人评价

SDAR 很干净：不让蒸馏与 RL 对抗，而是用门控让蒸馏成为支持的教练。这种非对称信任理念可能启发更广泛的 Agent 训练范式，比如多Agent 信用分配。
