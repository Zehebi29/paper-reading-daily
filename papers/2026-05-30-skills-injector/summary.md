# SkillsInjector: LLM Agent Skill Injection via Dynamic Context Construction

**arXiv**: 2605.29794 | **GitHub**: None (code promised upon publication) | **Affiliation**: Nanjing University

---

## One-line Summary

Injecting more skills into LLM agents doesn't always help and can hurt performance; SkillsInjector adaptively selects how many skills to inject and rewrites their descriptions, achieving 58.7% average pass rate across three benchmarks (vs 53.6% best baseline).

## Problem Context

LLM agents increasingly rely on skill libraries to handle complex tasks. But existing methods treat skill injection as a static process - fixed selection criteria, fixed budget, unchanged descriptions. In practice, which skills are injected, how many, and how they're presented all significantly affect agent performance. The core observation: semantically similar skills are not necessarily useful, and can actively hurt.

## Core Method

SkillsInjector is a two-stage adaptive framework:

- Context Planner: Trains an MLP scorer on frozen Qwen3-Embedding-0.6B to learn execution-grounded skill utility from actual task traces, rather than relying on semantic similarity. Uses adaptive budgeting - dynamically decides how many skills to inject per task instead of fixed top-K.
- Set-Aware Renderer: Distills Qwen3-235B rewriting capability into a Qwen3-8B small model, rewriting each skill description based on the current injection set to eliminate overlap and ambiguity (e.g., appending "Not for: refund_policy_reference").
- Training: Planner uses alignment loss + preference loss; Renderer uses curriculum learning, transitioning from trace-rich to trace-free rewriting tasks.

## Key Results

| Method | tau2-airline | tau2-retail | tau2-telecom | SkillsBench | ALFWorld | Average |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| No-skill | 37.6 | 51.2 | 40.0 | 5.2 | 67.1 | 40.2 |
| Best baseline (Graph of Skills) | 56.1 | 60.0 | 60.4 | 15.9 | 75.4 | 53.6 |
| SkillsInjector | 60.0 | 61.4 | 67.0 | 22.6 | 82.7 | 58.7 |

- Improvements of +3.9, +6.1, +7.3 pp over strongest baseline
- Ablation: Planner contributes most (removing it drops airline from 60.0 to 47.2); Renderer mainly reduces interaction overhead (removing it increases agent messages by 6-8)
- Compute: ~240 H200 GPU-hours total inference, ~24 H200 GPU-hours for training Planner + Renderer

## Highlights

1. Precise problem framing: Reframes skill injection as a per-task context construction problem rather than simple retrieval matching
2. Adaptive budgeting: Different tasks automatically decide how many skills to inject - some tasks perform better with zero skills, others need multiple
3. Set-aware rendering: Renderer dynamically adjusts descriptions based on the current injection set, avoiding ambiguity and competition between skills
4. Solid experiments: SOTA on 3 different domain benchmarks (customer service dialogue, code, text-based decision-making), with complete ablation studies

## Limitations

1. No open-source code: Paper promises code will be released upon publication but not yet available
2. Only tested on frozen agents: No exploration of interaction with RL-finetuned agents
3. Single skill source: Each benchmark uses only one type of skill library, no mixed-source testing
4. Only Qwen models tested: Not validated on GPT, Claude, or other closed-source models

## Personal Assessment

The core insight is very practical - the naive approach of stuffing more skills in does backfire in practice. SkillsInjector two components are well-designed: Planner handles how many and which ones, Renderer handles how to present them. The Renderer curriculum distillation from a large model to an 8B small model is particularly relevant for real-world deployment. For researchers working on agent skill systems, this paper is worth a close read, especially the analysis showing semantic similarity does not equal actual utility. The only downside is the lack of open-source code, but the experimental results are detailed enough to reproduce the core ideas.
