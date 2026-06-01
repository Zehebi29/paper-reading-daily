# Skill Reuse as Compression in Agentic RL (ReuseRL)

**arXiv**: 2605.31509 | **GitHub**: None (to be released) | **Institutions**: Arizona State University, University of Pennsylvania, University of Southern California

---

## One-sentence Summary

Formalizes agent skill reuse as a compression problem via the Minimum Description Length (MDL) principle, proposing ReuseRL which outperforms vanilla GRPO and pure round-length baselines across ALFWorld, TextWorld-Cooking, and Countdown.

## Problem Background

LLM agents trained with RL tend to learn brittle, task-specific shortcuts (reasoning collapse) that fail to generalize to OOD scenarios. The root cause is that agents learn incompressible trajectories -- each task uses a unique strategy with no reusable abstract structure. Existing round-length penalties only optimize for short, but short does not equal reusable, and can even cause catastrophic failures in some settings (e.g., TextWorld-Cooking).

## Core Methods

1. MDL Formalization: Defines trajectory compressibility as a two-part description length -- skill dictionary size plus segmentation cost to cover trajectories with the dictionary.
2. Skill Dictionary Extraction: Uses greedy BPE-style merging to extract shared skill subsequences online from successful trajectories (e.g., Take -> Transport -> Deliver in ALFWorld).
3. Segmentation Cost Penalty: Adds a SegCost term to the GRPO RL objective, penalizing trajectories that cannot be efficiently covered by the dictionary.
4. EM-style Optimization: E-step extracts optimal dictionary from current successful trajectories; M-step updates policy to prefer low segmentation cost trajectories.
5. Theoretical Guarantees: Proves pure round-length penalty is equivalent to using a singleton-only dictionary (cannot capture reusable structure), and provides a PAC-Bayes generalization bound.

## Key Results

| Method | ALFWorld IID | ALFWorld OOD | TW-Cooking | Countdown |
|--------|-------------|-------------|------------|-----------|
| Vanilla GRPO | 84.29% | 79.85% | 74.97% | 68.46% |
| Pure Round-Length | 96.43% | 91.79% | 64.03% | 77.02% |
| ReuseRL-SegCost | 97.14% | 93.28% | 81.73% | 80.37% |

Key findings:
- Pure round-length actually performs 10.94pp worse than GRPO on TW-Cooking, because it encourages irreversible mistakes (e.g., burning already-cooked ingredients)
- ReuseRL reduces burn failures in TW-Cooking from 74% to 3.3%
- Extracted skill dictionaries align well with environment structure

## Highlights

- Provides a clean theoretical framework (MDL) for agent skill reuse, not ad hoc engineering tricks
- The concept of compressible success is intuitive -- trajectories coverable by the dictionary are the ones that generalize
- The counterintuitive failure case analysis of pure round-length is compelling (short != good)
- PAC-Bayes generalization bound connects empirical results to theory

## Limitations

- Skill projection requires hand-crafted verb-mapping rules per environment, not generalizable
- Greedy BPE is approximate for large vocabularies (|Sigma| > 26)
- Only tested on 1.5B-1.7B small models, no validation on larger models
- Code not yet open-sourced

## Personal Assessment

The core insight of this paper is valuable: when training agents with RL, rewarding short trajectories and rewarding reusable trajectories are fundamentally different things. Pure round-length penalty is a crude form of compression -- it only considers length, not structure, and backfires in environments requiring multi-step compositional operations. ReuseRL formalizes structural compression via MDL, and this idea could extend to more scenarios.

However, practical reusability is questionable: the hand-crafted skill projection rules are the biggest bottleneck. If one could learn automatic skill abstractions (e.g., via VQ-VAE or tokenization approaches), it would be much more appealing. Also, experiments on 1.5B models are relatively small-scale -- it is unclear whether the gap between GRPO and ReuseRL remains as significant with 7B+ models, which may already have stronger implicit compression capabilities.
