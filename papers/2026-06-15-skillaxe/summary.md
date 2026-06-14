# SkillAxe: 通过评估引导的自优化打磨LLM编写的Agent技能

**arXiv**: 2606.10546 | **GitHub**: 无 | **机构**: Microsoft Research

---

## 一句话总结

SkillAxe 是一个无监督框架, 让LLM通过四维诊断(质量影响/触发精度/指令合规/路径覆盖)迭代修复自己写的agent技能, 在SkillsBench上将通过率提升28%, 在SpreadsheetBench上用22个技能就达到了LLM自生成69个技能的同等效果(52.0%).

## 问题背景

Agent技能(skill)是结构化的自然语言指令, 用来引导LLM执行工作流/验证约束/从失败中恢复. SkillsBench的基准测试显示: 人类编写的技能能让通过率提升16.2个百分点, 但LLM自己写的技能几乎没有任何提升. 核心矛盾在于: LLM能写语法顺顺的技能, 但在实际执行中完全不work.

## 核心方法

- **四维诊断框架**: 对每个技能s, 分别在有/无技能注入的情况下运行agent, 从四个维度诊断差异
- **质量影响(Quality Impact)**: LLM judge比较有/无技能的输出偏好方向和改进幅度, 输出[-1, 1]分数
- **触发精度(Trigger Precision)**: 在embedding空间中分析技能的触发短语, 用CoverageBreadth/NegativeSpecificity/BoundarySharpness三个指标衡量
- **指令合规+故障归因(Instruction Compliance)**: 将技能拆成规则R_i, 对每条规则评估adherence/quality/fault, 区分是技能写得差还是agent执行差
- **路径覆盖(Solution-Path Coverage)**: 枚举任务的可行路径, 用cosine similarity衡量技能是否覆盖了合理的执行策略
- **迭代优化**: 基于诊断结果生成结构化改进brief, 无需ground truth标签或测试套件

## 关键结果

**SkillsBench (77个任务):**

| 条件 | Coverage | Fair Quality | 整体通过率 |
|:---|:---|:---|:---|
| 无技能 | 46.7% | 91.4% | 42.7% |
| LLM自写(未优化) | 49.4% | 78.9% | 39.0% |
| **SkillAxe** | **72.7%** | **75.0%** | **54.5%** |
| 人类编写 | 68.8% | 90.6% | 62.3% |

- SkillAxe的通过率比未优化LLM技能提升28%相对值(39.0% -> 54.5%)
- 缩小了与人类编写技能47-67%的差距
- 主要贡献在执行可靠性(coverage从46.7%->72.7%), 而非答案质量

**SpreadsheetBench (200训练/50测试):**

| 条件 | 通过率 | 技能数 | 激活率 |
|:---|:---|:---|:---|
| 无技能 | 16.0% | - | - |
| LLM自写库 | 52.0% | 69 | 20.0% |
| **SkillAxe库** | **52.0%** | **22** | **35.8%** |

- 同等准确率下技能数量减少68%, 激活率近乎翻倍

## 亮点

- 四维诊断设计精巧, 每个维度都有明确的可量化指标, 不依赖ground truth
- 故障归因机制(skill fault vs agent fault)避免了“修技能反而改坏”的问题
- 在SpreadsheetBench上验证了作为持续学习引擎的实用性, 22个技能就能达到69个的效果
- 发现了关键洞察: 技能的主要价值是编码程序性知识(库使用/格式处理/工作流结构), 防止脆弱的执行失败

## 局限

- 没有开源代码, 9页短文, under review状态
- SkillsBench的任务覆盖范围有限(77个任务/11个领域), 真实场景可能更复杂
- 四维诊断依赖LLM judge, 存在judge本身的偏见和不稳定性
- 没有和其他技能优化方法(SkillRevise/CoEvoSkills/SkillRL)做直接对比

## 个人评价

这篇论文的核心洞察很实在: LLM写的技能不是“写得差”, 而是“写得不对路” -- 语法完美但执行时完全不匹配agent的实际行为模式. 四维诊断框架的设计思路值得借鉴, 特别是故障归因(skill fault vs agent fault)这一维度, 在实际部署中非常实用. 对于做agent skill系统的人来说, 这个“写-测-诊断-修”的闭环比一次性生成靠谱得多. 不过没开源代码有点可惜, 方法的很多细节(比如embedding空间的trigger分析怎么做/LLM judge的prompt怎么设计)只能从论文猜.
