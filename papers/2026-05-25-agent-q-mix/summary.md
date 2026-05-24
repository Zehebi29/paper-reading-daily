# Agent Q-Mix: Selecting the Right Action for LLM Multi-Agent Systems through Reinforcement Learning

**arXiv**: 2604.00344 | **GitHub**: https://github.com/ericjiang18/Agent-Q-Mix | **Institution**: Individual/Independent Research

---

## One-Sentence Summary

Agent Q-Mix uses QMIX multi-agent reinforcement learning to learn optimal communication topologies for LLM multi-agent systems, achieving 72.73% average accuracy across 7 benchmarks, outperforming AutoGen by 4.13 points while using fewer tokens.

## Problem Background

In LLM multi-agent systems, the communication topology (who talks to whom) directly affects collaboration quality. Existing methods either use static topologies (fully connected, chain) or centralized topology generators, preventing individual agents from adapting their communication behavior based on their local state and task difficulty.

## Core Method

1. **Communication action space**: Each agent chooses from 6 discrete actions - solo_process (work independently), broadcast_all (broadcast), selective_query (targeted query), aggregate_refine (gather and refine), execute_verify (execute and verify), debate_check (adversarial debate) - collectively determining the round-wise communication graph structure.
2. **Topology-aware Q-Network**: GNN (2 layers, hidden 128) encodes the current communication graph, GRU (hidden 128) models temporal dependencies, MLP outputs per-action Q-values. All agents share network parameters.
3. **QMIX monotonic value factorization**: Enables CTDE (centralized training, decentralized execution), ensuring each agent can independently select argmax actions without conflict. Reward combines accuracy (positive) with normalized token usage penalty.
4. **Multi-round execution**: Each round: select communication actions -> construct adjacency matrix -> execute in topological order -> aggregate. 3 rounds for math, 2 for coding/reasoning.

## Key Results

On GPT-OSS:120B, Agent Q-Mix achieves **72.73% avg accuracy** vs AutoGen 68.60% and single-agent 48.73%. On Humanity's Last Exam (HLE), achieves 20.8% vs Microsoft Agent Framework 19.2%.

Full results table (7 benchmarks):

| Model | LiveCode | HumanEval | MMLU-Pro | AIME25 | AIME26 | HMMT | Beyond | **Avg** |
|-------|----------|-----------|----------|--------|--------|------|--------|---------|
| Single-agent | 88.75 | 90.04 | 74.29 | 23.33 | 20.00 | 26.67 | 18.00 | **48.73** |
| AutoGen | 100.00 | 95.12 | 81.43 | 53.33 | 70.00 | 43.33 | 37.00 | **68.60** |
| **Agent Q-Mix** | **100.00** | **97.56** | **92.86** | **63.33** | 60.00 | **53.33** | **42.00** | **72.73** |

## Highlights

1. Action space design is elegant: 6 actions cover almost all common communication patterns with clear graph-theoretic semantics, making it highly interpretable.
2. CTDE + QMIX is a natural fit: monotonic value factorization guarantees optimal decentralized execution, and the GNN encoder lets agents perceive the current communication graph structure.
3. Training efficiency is impressive: only 15 examples per domain, 50 episodes, 30-60 minutes on a single CPU.
4. Strong robustness: topology automatically adapts when agents fail, outperforming other frameworks in fault scenarios.

## Limitations

1. Only controls topology structure, not message content itself - may miss content-level collaboration gains.
2. Extremely small training data (15 examples + 50 episodes) may lead to local optima in complex scenarios.
3. Does not fine-tune LLM parameters, so performance is bounded by the base model capability.

## Personal Assessment

What impresses me most is the clean perspective shift of reframing communication topology selection as a MARL problem. Existing multi-agent systems mostly rely on fixed topologies or heuristic rules. Agent Q-Mix lets each agent "decide for itself who to talk to," significantly outperforming commercial frameworks like AutoGen on math reasoning and coding tasks.

For GWS experiments, if we eventually work on multi-robot weld path planning collaboration, this adaptive topology idea is highly inspiring - each welding robot could autonomously decide whether to plan independently, exchange path information, or request aggregation. However, the paper only validates text-based communication between LLMs; applying it to real robot collaboration would require solving perception-layer and physical-layer communication challenges.

Also noteworthy is the training efficiency: 15 examples + short training + strong results - a pattern worth learning for tight submission schedules.
