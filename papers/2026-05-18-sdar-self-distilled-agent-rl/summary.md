# Self-Distilled Agentic Reinforcement Learning (SDAR)

> arXiv: 2605.15155v1 | 2026-05-14 | cs.LG / cs.AI / cs.CL
> Code: https://github.com/ZJU-REAL/SDAR

## Problem

RL-trained LLM agents suffer from sparse trajectory-level rewards in multi-turn tasks. On-Policy Self-Distillation (OPSD) adds token-level dense supervision via a teacher model with privileged context, but naively combining GRPO+OPSD causes catastrophic failure in multi-turn agents.

## Method: SDAR = GRPO + Gated OPSD

1. RL backbone: GRPO as primary optimization objective
2. OPSD auxiliary: Teacher with privileged context computes token-level log-probability gap
3. Sigmoid gating (core): Adaptive gate modulates distillation per token. Positive gaps trusted, negative gaps attenuated. Gate detached from gradient flow.
4. Skill retrieval: UCB bandit or keyword matching from SkillBank

## Results

| Benchmark | vs GRPO |
|:---|---:|
| ALFWorld | +9.4% |
| Search-QA | +7.0% |
| WebShop-Acc | +10.2% |
| WebShop-Reward | +5.2 |

Models: Qwen2.5-3B/7B, Qwen3-1.7B. Open source: github.com/ZJU-REAL/SDAR

## Strengths

- Elegant: each token decides its own supervision intensity
- Asymmetric trust: positive gap = endorsement; negative gap may not be rejection
- Multi-model validation, code available
- Gating stabilizes RL training

## Limitations

- Requires SkillBank and retrieval module
- Task-dependent privileged context design
- Text-only environments; no vision/tool-use settings
- Small models only (1.7B-7B); large-model scaling unknown

## Personal Take

SDAR is clean: do not let distillation fight RL; use gating to make distillation a supportive coach. The asymmetric trust philosophy could inspire broader agent training paradigms like multi-agent credit assignment.
