# Orchard: An Open-Source Agentic Modeling Framework

**arXiv**: 2605.15040 | **GitHub**: https://github.com/microsoft/Orchard | **Microsoft Research**

---

## One Sentence Summary

Microsoft open-sourced Orchard, an end-to-end agent training framework with K8s-native environment service and three recipes (SWE/GUI/Claw), achieving 67.5% on SWE-bench Verified (30B model -- new SOTA among comparable open models).

## Problem Background

High-performance AI agent systems rely on proprietary codebases. Open-source frameworks focus on orchestration, lacking scalable training infrastructure.

## Core Method

### Orchard Env
Three-layer: Client SDK -> Orchestrator (FastAPI on K8s) -> In-Pod Agent. Key: Init Container agent injection, Pod-IP direct connect.
Cost: 128 sandboxes x 240h -> on-demand $3,362, spot $673 (vs E2B $7,078, Modal $10,305). Latency: 0.28s avg.

### Orchard-SWE
Qwen3-30B-A3B-Thinking. SFT: 64.3%. SFT+RL (BAR): 67.5%. Key: credit-assignment SFT from failed trajectories.

### Orchard-GUI
Qwen3-VL-4B-Thinking. SFT: 52.0% avg. SFT+RL: 68.4% avg (WebVoyager 74.1%). RL adds +16.4 pts.

### Orchard-Claw
192 synthetic tasks, SFT+GRPO. Claw-Eval pass@3: 59.6%.

## Highlights
1. True open-source training framework (SFT+RL)
2. Orchard Env: Init Container injection, cost 1/10 commercial
3. Learning from failure: credit-assignment SFT
4. Cross-domain: SWE/GUI/Claw unified

## Limitations
1. 30B model scale
2. Claw only 192 tasks
3. RL training complexity
4. K8s dependency

## Personal Thoughts
Contrasts with ToolCUA: ToolCUA focuses on GUI RL pipeline, Orchard on infrastructure. The Init Container injection is very clever -- using native K8s capability for agent training.
