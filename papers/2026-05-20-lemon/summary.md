# LEMON: Learning Executable Multi-Agent Orchestration via Counterfactual RL

**arXiv**: 2605.14483 | **Code**: https://anonymous.4open.science/r/LEMON-B23C | **Org**: Anonymous (NeurIPS 2026 under review)

---

## TL;DR

LEMON trains an LLM orchestrator to output a complete YAML spec (roles + capacities + dependencies) for multi-agent systems. Key innovation: localized counterfactual credit assignment -- mutate one field, execute counterfactual, backprop reward delta only to edited token spans. 90.72% avg accuracy across 6 benchmarks, SOTA among multi-agent orchestration methods.

## Problem

Multi-agent LLM systems depend on orchestration design: which roles, what model tier per agent, how they connect. Existing methods optimize these separately (topology OR workflow OR routing) with sparse end-to-end rewards that cannot pinpoint which local decision caused success/failure.

## Method

1. Compositional orchestration generation: Orchestrator (Qwen2.5-7B) outputs YAML with agent roles, capacity levels (small/medium/large = 7B/14B/32B), and dependency refs. Compiled to DAG.

2. SFT warmup: Teacher model generates valid YAML examples for supervised pretraining to avoid invalid RL rollouts.

3. Orchestration-level GRPO: Reward = task correctness + token efficiency bonus - graph complexity penalty. Sparse credit.

4. Localized counterfactual credit (core): Three mutation types -- delete dependency, rollback role to base, downgrade capacity. Execute counterfactual spec, compute reward delta, backprop ONLY to edited token spans. Not the whole sequence.

5. Adaptive mutation sampling: Online balance of three mutation types based on historical reward contrast.

## Results

Six benchmarks:

| Benchmark | Size | Type |
|:----------|:-----|:-----|
| MMLU (val) | 1,531 | MCQ reasoning |
| GSM8K (test) | 1,319 | Grade-school math |
| AQuA | 254 | Math reasoning |
| MultiArith | 600 | Multi-step arithmetic |
| SVAMP | 1,000 | Math word problems |
| HumanEval | 124 | Code generation |

Average accuracy: 90.72%. Outperforms single-agent (Vanilla/CoT/SC), fixed-topology MAS (Chain/Tree/Complete), adaptive workflow (AFlow), and topology design methods (G-Designer, AgentPrune, etc.). Pareto-optimal on accuracy-token efficiency.

Ablations: removing SFT -> many invalid rollouts early in RL; removing local CF -> significant performance drop; execution caching reduces counterfactual overhead.

## Strengths

- Credit granularity: from trajectory-level to span-level, directly solving the multi-agent RL credit assignment bottleneck
- Unified spec: roles + capacities + dependencies co-optimized, not sequential
- Practical: SFT + RL with 1500 steps, group size 4, manageable compute
- Heterogeneous workers: 7B/14B/32B mixed, realistic deployment setup
- Three mutation types are well-designed, directly mapping to orchestration dimensions

## Limitations

- Code on anonymous 4open.science (double-blind), reproducibility unverified
- Benchmarks are reasoning/coding only; no tool-use, multi-turn interaction, or open-domain tasks
- Fixed orchestrator backbone (Qwen2.5-7B), stronger backbones unexplored
- Counterfactual evaluation doubles training cost vs vanilla GRPO (partially mitigated by caching)
- Static orchestration (generated once before execution), dynamic re-orchestration not explored
- Only trains orchestrator, not workers: orchestration + execution joint RL unknown

## Personal Take

LEMON tackles a practical pain point: automating multi-agent system design. Current practice is manual topology tuning + gut-feel role assignment + uniform strongest model = expensive and suboptimal. LEMON's answer: make orchestration design a differentiable generation problem with counterfactual edits as localized credit signal.

The three mutation types (delete dependency / rollback role / downgrade capacity) show deep domain understanding -- not random perturbations but surgical edits on the three real orchestration axes.

Open question: if we also fine-tune worker LLMs (not just the orchestrator), could joint "orchestration + execution" RL beat separate training? LEMON trains only the orchestrator, which simplifies but caps potential. Also, LEMON's orchestration is static (generated once pre-execution); dynamic re-orchestration mid-task could be the next breakthrough.
