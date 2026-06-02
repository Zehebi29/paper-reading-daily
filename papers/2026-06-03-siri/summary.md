# SIRI: Self-Internalizing Reinforcement Learning with Intrinsic Skills for LLM Agent Training

**arXiv**: 2606.02355 | **GitHub**: https://github.com/kirito618/SIRI | **Institutions**: Xiamen University, Meituan, Macao Polytechnic University

---

## One-sentence Summary

Proposes a three-phase RL framework where LLM agents discover, validate, and internalize reusable skills entirely from their own rollouts, then discard the skill bank at inference - achieving 0.930 on ALFWorld and 0.813 WebShop success with Qwen2.5-7B, outperforming SkillRL which requires persistent skill retrieval.

## Problem Background

Skill-based RL methods for LLM agents typically depend on (1) external skill generators that distill trajectories into reusable skills, and (2) persistent skill banks that must be retrieved at inference time. Both introduce engineering complexity, increase context length and latency, and create deployment bottlenecks. The core question: can an agent discover skills by itself during training and absorb them into its own parameters so that no skill infrastructure is needed at deployment?

## Core Methods

1. **Phase 0 - Policy Warmup (GiGPO)**: Bootstraps basic interaction ability using Group-in-Group Policy Optimization, which combines episode-level and step-level advantages. Collects successful skill-free trajectories for later mining. Adaptive transition when enough successful experience is gathered (N_succ >= N_req).

2. **Phase 1 - Self-Skill Mining and Validation**: Every I_mine iterations, a frozen snapshot of the policy summarizes successful trajectories into condition-strategy skill pairs. Paired rollouts (skill-augmented vs. skill-free) estimate each skill utility via treatment effect (Delta_g = R_g_skill - R_g_base). Skills are promoted to active if utility exceeds threshold after N_eval retrievals; persistently negative skills are retired.

3. **Phase 2 - Advantage-Weighted Skill Internalization**: Only beneficial skill-guided tokens are distilled into the plain policy. A composite utility gate filters by: (a) skill was used, (b) skill had positive utility, (c) position is an action token. Distillation weights are scaled by clipped RL advantage. Joint loss: L_GiGPO + beta*L_KL + lambda(k)*L_int with linear warmup of lambda.

4. **Key Design Principle**: Skills are temporary training-time signals, not permanent inference-time dependencies. After training, the skill bank is discarded - inference uses only the original prompt with no retrieval service or external memory.

## Key Results

| Method | ALFWorld All | WebShop Score | WebShop Success |
|--------|-------------|---------------|-----------------|
| ReAct | 0.312 | 0.462 | 0.195 |
| Reflexion | 0.427 | 0.581 | 0.288 |
| PPO | 0.804 | 0.814 | 0.687 |
| GRPO | 0.776 | 0.793 | 0.661 |
| GiGPO | 0.908 | 0.844 | 0.728 |
| SkillRL | 0.899 | 0.852 | 0.727 |
| **Siri** | **0.930** | **0.899** | **0.813** |

Key findings:
- Siri improves GiGPO by +2.2pp on ALFWorld and +8.5pp on WebShop success
- Outperforms SkillRL (which uses persistent skill retrieval) while requiring zero inference-time skill infrastructure
- Ablation: removing Phase 0 warmup causes -10.2% success drop, confirming that mining skills from an untrained policy injects noise
- Skills are compact and interpretable - learned skill descriptions align with environment structure

## Highlights

- Elegant train-with-skills, deploy-without-skills paradigm eliminates the skill retrieval bottleneck entirely
- Self-skill mining from the agent own rollouts avoids dependency on external trajectory distillers or teacher models
- Paired validation with treatment effect estimation is a principled way to filter spurious skills
- The three-phase progression (warmup -> mining -> internalization) is well-motivated and each phase contribution is cleanly ablated

## Limitations

- Tested only on Qwen2.5-7B-Instruct - unclear if the skill internalization advantage holds for larger models that may already have stronger implicit skill compression
- ALFWorld and WebShop are relatively structured environments; more open-ended tasks (web browsing, coding) may challenge the condition-skill pair format
- The I_mine interval and N_eval thresholds introduce hyperparameters that need tuning per environment
- No comparison with ReuseRL (MDL-based skill compression), which addresses a similar problem from a different angle

## Personal Assessment

SIRI and ReuseRL (covered yesterday) tackle the same fundamental tension in agent RL: how to get the benefits of skill reuse without paying the inference-time cost. Their approaches are complementary - ReuseRL uses MDL compression to prefer compressible trajectories in the reward signal, while SIRI explicitly mines skills and distills them back into the model weights. SIRI approach is more direct (you literally internalize the skills) but requires the three-phase pipeline; ReuseRL is more elegant theoretically but the hand-crafted skill projection is a bottleneck.

The most interesting aspect is that SIRI achieves better results than SkillRL *without* using skills at inference time. This suggests that the skill-augmented training signal acts as a form of implicit regularization or curriculum, guiding the policy toward more structured solutions that it can then execute without the scaffolding. This is analogous to how humans learn: you practice with training wheels, then ride without them - the skill was in the learning, not the equipment.

The practical implication is significant: if you can get skill-level performance without deployment complexity, the cost-benefit tradeoff for skill-based training shifts dramatically. The open question is whether this scales to environments where skills are not discrete strategy descriptions but continuous behaviors.
