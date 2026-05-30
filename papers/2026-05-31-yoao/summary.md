# You Only Align Once: Propagating Cooperative Behaviors in Multi-Agent Systems through Seed Agents

**arXiv**: 2605.27586 | **GitHub**: https://github.com/arcarae/YOAO | **Affiliation**: Arcarae, UC Santa Cruz, Northwestern, Johns Hopkins

---

## One-line Summary

A single fine-tuned seed agent can propagate cooperative behaviors to untrained agents purely through natural-language interaction, doubling cooperation rates from 24.8% to 62.2% in a team-based dilemma and achieving 91.5% zero-shot trade success in a spatial survival simulation versus 21.6% baseline.

## Problem Context

Multi-agent systems need reliable alignment, but aligning every agent individually does not scale to open environments with many interacting models. Existing approaches either rely on per-agent RLHF/DPO or prompt instructions, both of which have limited effectiveness. Prompts specify what to do but cannot teach agents how to persuade others to cooperate in adversarial environments.

## Core Method

- Alignment Propagation: Fine-tune only one seed agent and let cooperative behavior spread to untrained agents through natural-language interaction
- Seed Training: Use Kimi-K2 as teacher to generate 10000 game trajectories, then LoRA fine-tune Qwen-3-14B (rank=128, alpha=256)
- Two Test Environments: Red-Black Game (team Prisoners Dilemma with broadcast deliberation) and Sugarscape (spatial grid with pairwise trading)
- Mechanism Validation: Mute test proves propagation depends on semantic persuasion, not mere presence

## Key Results

| Experiment | Metric | Result |
|------------|--------|--------|
| Red-Black Game (1 seed) | Cooperation rate | 24.8% to 62.2% (2.5x) |
| Red-Black Game (4 seeds) | Cooperation rate | 95.6% |
| Sugarscape (zero-shot) | Trade success | 21.6% to 91.5% (4.2x) |
| Sugarscape (zero-shot) | Survival rate | 13% to 85% |
| Cross-arch (Qwen to LLaMA/Mistral) | Cooperation | 85%+ with 3+ seeds |
| Large groups (N=10+) | Seed threshold | 20% seeds yields 98-100% cooperation |

## Highlights

- Only one seed agent needs training, greatly reducing multi-agent alignment cost
- Zero-shot cross-environment transfer: Red-Black Game seeds work on Sugarscape directly
- Cross-architecture propagation: Qwen-trained seeds influence LLaMA and Mistral
- Clear causal analysis via mute test

## Limitations

- Propagation quality bounded by target model receptiveness; some models need more seeds
- Broadcast communication far more efficient than pairwise (20% vs 50% seed threshold)
- Experiments use simplified game environments; complex tasks not yet tested
- Cooperation persists after seed removal (82.5%) but decays over time

## Personal Take

The core insight is fascinating: alignment does not need to cover every agent, you just need one missionary. This mirrors how norms propagate in human societies through influential individuals rather than universal training.

Practically, deploying multi-agent systems could only require alignment of key roles like orchestrators, with other agents learning cooperation through interaction. Good for cost and scalability.

However, the experimental environments are relatively idealized. Whether persuasion-based propagation works in complex scenarios like multi-step tool use or code collaboration needs further validation. Cooperation decays after seed removal, suggesting long-running systems need continuous seed maintenance.
