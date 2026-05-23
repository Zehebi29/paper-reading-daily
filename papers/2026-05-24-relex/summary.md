# RELEX: Just 15% RLVR Training Enough for Full Performance

**arXiv**: 2605.21468 | **GitHub**: https://github.com/weizhepei/RELEX | **Affiliation**: UVa & Washington University in St. Louis

---

## One-Sentence Summary

RELEX discovers that RLVR weight trajectories are naturally rank-1 and linearly predictable.
By observing only the first 15% of training steps, it extrapolates to full training performance,
matching or exceeding full RLVR training on three Qwen models.

## Background

RLVR has become the dominant paradigm for improving LLM reasoning (e.g., DeepSeek-R1 GRPO),
but full training is computationally expensive. The geometry of training dynamics was previously underexplored.

## Method

- Finding 1: RLVR weight trajectories are low-rank -- per-tensor deltas approximated by rank-1 SVD
- Finding 2: Rank-1 coefficients evolve near-linearly (R^2 > 0.98)
- RELEX algorithm: deltas from early checkpoints -> SVD rank-1 subspace -> linear fit -> extrapolate
- Zero training cost: closed-form truncated SVD + two-parameter least-squares fit

## Key Results

### In-domain MATH (15-20% observation, extrapolate to 500 steps)

| Model | Base | Full RLVR | RELEX |
|-------|:----:|:---------:|:-----:|
| Qwen2.5-Math-1.5B | 48.2 | 71.5 | **71.6** |
| Qwen3-4B-Base | 64.0 | 85.5 | **85.6** |
| Qwen3-8B-Base | 73.9 | 88.5 | 87.4 |

### OOD Benchmarks (average of 5)

| Model | Full RLVR | RELEX |
|-------|:---------:|:-----:|
| Qwen2.5-Math-1.5B | 28.4 | **30.0** |
| Qwen3-4B-Base | 42.3 | **43.0** |
| Qwen3-8B-Base | 47.1 | 46.2 |

### vs Baselines (15-20% cost)

RELEX outperforms ExPO, AlphaRL, Logits Extrapolation, and Weight Extrapolation.

### Extrapolation Limit

- Observing 50 steps, extrapolates to 1000 steps with continued improvement
- Stable up to 10-20x beyond observation window

## Highlights

- Extreme simplicity: rank-1 + linear fit suffices; rank-5/10 offers no gain
- Valuable theoretical insight: RLVR optimization follows straight-line trajectory
- Practical: open-source code + HuggingFace checkpoints released
- Denoising effect: rank-1 projection discards optimization noise

## Limitations

- Only validated on math reasoning tasks (MATH/AIME/AMC), not code, agent, or general chat
- Requires existing RLVR training trajectory as starting point
- Observation window selection is sensitive

## Personal Take

The counter-intuitive finding is the most exciting part: everyone assumes RLVR pushes models
into complex high-dimensional spaces, but the trajectory is nearly a straight line. This could
mean Agent RL training (like GWS path planning RL) might have similar low-rank structure.

The denoising perspective is also interesting for GWS reward curve analysis.
