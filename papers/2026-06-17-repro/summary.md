# RePro: Retrospective Progress-Aware Self-Refinement for LLM Agent Training

**arXiv**: 2606.14302 | **GitHub**: None | **机构**: Shanghai Jiao Tong University (SJTU)

---

## One-line Summary

RePro trains LLM agents to self-generate step-wise progress signals via a forward-then-reflect paradigm, achieving up to +17.5% absolute success rate gains on ALFWorld and WebShop without any external progress supervision at RL time.

## Problem Background

LLM agents trained with RL optimize step-wise action prediction but lack metacognitive awareness of task progress -- a gap that hinders long-horizon scaling. A pilot study reveals a critical asymmetry: online progress prompting (self-assessing during execution) hurts performance by -8.6% avg, while retrospective demonstrations (assessing after knowing the outcome) help by +7.9% avg. This means progress awareness is beneficial but cannot be learned from outcome-reward training alone or forced through prompting.

## Core Method

- **Forward-then-Reflect Paradigm**: Agent executes actions online generating progress estimates, then after trajectory completion retrospectively reassesses step-wise progress anchored by the known outcome.
- **Phase 1 - Retrospection Warmup**: SFT on successful trajectories annotated with progress sequences by an external LLM (DeepSeek-V4), teaching the agent the reflection format.
- **Phase 2 - RePro-PO**: RL with composite reward: environment reward + progress shaping reward (beta * delta progress) + online-retrospective alignment reward (penalizes divergence between online and retrospective estimates) + format regularization.
- **Hierarchical Advantage**: Uses GiGPO-style episode-level + step-level advantages for fine-grained credit assignment.
- **Self-Generated Signals**: After warmup, the agent generates its own progress annotations during RL training, eliminating the need for continuous external supervision.

## Key Results

| Environment | Model | RePro SR | Meta Prompt SR | Absolute Gain |
|-------------|-------|----------|----------------|---------------|
| WebShop | Qwen2.5-1.5B | 54.53 | 45.55 | +8.98 |
| WebShop | Qwen2.5-3B | 56.14 | 44.57 | +11.57 |
| WebShop | Qwen2.5-7B | 57.43 | 51.61 | +5.82 |
| ALFWorld | Qwen2.5-1.5B | 62.50 | 44.96 | +17.54 |
| ALFWorld | Qwen2.5-7B | 74.65 | 69.96 | +4.69 |
| Sokoban | Qwen3-4B | 38.96 | 31.38 | +7.58 |

- Pilot study: DeepSeek-V4 on WebShop drops from 35.9% to 33.6% with online prompting, rises to 43.5% with retrospective demos.
- GPT-5.1 on WebShop rises from 44.1% to 51.6% with retrospective demonstrations.
- Progress prediction discrimination (ability to distinguish success vs failure trajectories): baseline -1.07 to 1.86, RePro 6.37 (1.5B) / 31.62 (3B) / 11.14 (7B).

## Highlights

1. Elegant insight: progress awareness is beneficial but online prompting hurts; retrospective assessment after knowing outcome is the right training signal.
2. Two-phase design is clean -- warmup teaches format, RL-PO refines the skill with self-generated signals.
3. Composite reward is well-motivated: progress shaping + online-retrospective alignment + format regularization, each addressing a distinct failure mode.
4. Ablation is thorough: warmup-only catastrophically fails, warmup+GiGPO (no retrospective mechanism) underperforms by 5-6%, confirming the forward-then-reflect mechanism is essential.

## Limitations

1. No public code released yet, making reproduction difficult.
2. Benchmarks are relatively simple (WebShop, ALFWorld, Sokoban) -- unclear how well this scales to more complex real-world agent tasks.
3. Relies on external LLM (DeepSeek-V4) for warmup annotation, adding cost and dependency.
4. The method adds training complexity (two-phase training, composite reward with multiple hyperparameters alpha/beta).

## Personal Evaluation

This paper addresses a genuine gap in agentic RL training. The key insight -- that progress awareness must be learned retrospectively rather than prompted online -- is non-obvious and well-validated through the pilot study. The forward-then-reflect paradigm is conceptually clean and the composite reward design shows careful engineering thinking.

What I find most interesting is the connection to how humans develop metacognitive skills: we learn to assess our progress by reflecting on completed tasks, not by trying to predict progress in real-time. The paper formalizes this intuition effectively.

The practical limitation is the benchmark scale. WebShop/ALFWorld/Sokoban are standard but relatively constrained environments. It would be compelling to see this applied to longer-horizon tasks like SWE-bench or multi-step tool use where progress awareness matters more. The reliance on external LLMs for warmup annotation also limits scalability, though the paper shows the agent can self-generate signals after warmup.

For anyone working on agent RL training, the pilot study findings alone (online prompting hurts, retrospective helps) are worth knowing -- this is a common pitfall that many practitioners might fall into.
