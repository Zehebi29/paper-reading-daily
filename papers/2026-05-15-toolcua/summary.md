# ToolCUA: Towards Optimal GUI-Tool Path Orchestration for Computer Use Agents

> arXiv:2605.12481v1 · 2026-05-12 · [GitHub](https://github.com/X-PLUG/ToolCUA)

## 问题
计算机使用Agent面对混合动作空间（GUI操作 vs 工具调用）时，出现两种失效模式：
- **工具使用不足**（Tool Underuse）：几乎不调工具，纯GUI操作，效率低
- **工具滥用**（Tool Overuse）：过度调用工具，反而损害准确率

## 方法
三阶段训练范式：

1. **交错GUI-Tool轨迹生成**：将已有纯GUI轨迹（1万条）转化为GUI+工具混合轨迹，共生成18万步
2. **工具引导的GUI RFT**：在关键决策点（GUI↔Tool切换边界）上用GRPO做强化学习
3. **在线Agentic RL**：用路径效率奖励（Tool-Efficient Path Reward）训练，鼓励最优动作路径

核心奖励函数：`R = R_fmt + R_acc + λ·R_tool + β·R_length`

## 结果

| 指标 | 数值 |
|:----|:----:|
| OSWorld-MCP准确率 | **46.85%**（+66% vs 基线） |
| 参数量 | 8B |
| 平均完成步数 | **14.9**（所有模型中最少） |
| 对比Claude-4.5-Sonnet | 仅差1.5%（397B vs 8B） |

## 个人判断
这篇论文的优势在于：不靠更大的模型，而是靠更聪明的训练策略解决了Agent行为优化问题。三阶段设计层层递进，每个阶段解决一个具体痛点。开源代码也降低了复现门槛。

**关键词**：GUI Agent, RL Training, Hybrid Action Space, Tool Orchestration
