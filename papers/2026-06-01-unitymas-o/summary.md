# UnityMAS-O: A General RL Optimization Framework for LLM-Based Multi-Agent Systems

**arXiv**: 2605.26646 | **GitHub**: https://github.com/chenyiqun/UnityMAS-O

---

## one-sentence summary

UnityMAS-O treats the entire multi-agent workflow as the RL optimization unit, supporting joint optimization of arbitrary MAS topologies via a unified interface, achieving 3x improvement on code generation (0.255 to 0.686) and substantial gains on search QA tasks.

## problem background

LLM-based multi-agent systems can solve complex tasks through role decomposition (planner/retriever/coder etc.), but most remain manually orchestrated: workflows are statically specified by prompts, tools, and control rules, while agents themselves are rarely optimized via RL. Existing RL post-training frameworks (TRL, OpenRLHF, verl) only support single-policy optimization, lacking abstractions for structured interaction, role-level credit assignment, and configurable parameter sharing in multi-agent workflows.

## core methods

1. **Workflow-as-optimization-unit**: Treats the user-defined multi-agent workflow as a whole trainable object, rather than optimizing a single response or policy trajectory
2. **Four first-class abstractions**: Logical Agent Roles, Graph-Structured Trajectories, User-Defined Reward Functions (role/turn/trajectory three-level rewards), Explicit Agent-Model Mappings (logical role to physical model mapping)
3. **Three parameter sharing modes**: Supports full sharing (all roles share one model), full separation (each role has independent model), partial sharing, flexibly switchable within one interface
4. **Star-topology runtime**: Central controller executes workflow and assembles rewards, model-local worker groups handle rollout, advantage computation and PPO updates, heavy tensors stay local to reduce communication
5. **Role-level credit assignment**: Supports node-level, turn-level, trajectory-level three granularities of reward allocation, including delta rewards (e.g., verifier score improvement)

## key results

| Task | Model Config | Metric | Before | After | Improvement |
|------|-------------|--------|--------|-------|-------------|
| Reflective Code (TACO) | 3xQwen3-4B (separated) | all-passed | 0.255 | 0.686 | +169% |
| Reflective Code (TACO) | 3xQwen3-8B (separated) | all-passed | 0.290 | 0.738 | +154% |
| Reflective Code (TACO) | 3xQwen3-4B (separated) | avg verify turns | ~2.5 | ~1.7 | -32% |
| M-ASK Search (HotpotQA) | Shared 3B | F1 | - | 0.520 | close to independent 0.529 |

- QA/Search domain shows improvements across all workflows and model scales, small models (0.5B) achieve ~1000-2000% relative improvement
- M-ASK iterative search excels on multi-hop HotpotQA
- Shared parameter scheme (shared M-ASK 3B) reaches 0.520 F1, close to independent model 0.529, proving parameter sharing is viable

## highlights

- Precise abstraction design: separates workflow graph, role-model mapping, reward interface into independently configurable objects, truly task-agnostic
- Solid systems engineering: built on verl + Ray, supports multi-node distributed training, heavy tensor localization reduces communication overhead
- Comprehensive evaluation: covers search QA (Parallel Retrieval / Retrieve-Extract-Answer / M-ASK) and code generation (Reflective Verification Loop), spanning 0.5B-8B model scales
- Open-source code with thorough documentation, GitHub includes complete workflow configuration examples and extension guides

## limitations

- Only validated on Qwen model series, cross-model-family generalization not discussed
- High computational cost: requires multiple GPUs/nodes running multiple model instances simultaneously, though partial sharing saves resources, no specific GPU hours reported
- Evaluation tasks mainly concentrated on QA and code generation, more complex agent scenarios (web interaction, SWE-bench) still in planned stage

## personal take

This work addresses a very practical pain point in multi-agent RL training: existing RL frameworks are designed for single-policy optimization, and end-to-end training of multi-agent systems requires extensive custom code. UnityMAS-O core insight is treating the workflow itself as an optimizable object, decoupling logical roles from physical models through the mapping function -- this design is much more flexible than hard-coded multi-agent RL.

From a practical standpoint, the 3x improvement on code generation tasks is very significant, especially on TACO-Verified which has a standard test suite. The delta reward design (reflector is rewarded for improvement amount rather than absolute score) is also reasonable, avoiding reward hacking between verifier and coder.

However, note that this framework current positioning is more like a MAS RL training infrastructure rather than a plug-and-play agent framework. Extending new workflows requires writing WorkflowRunner and RewardAllocator plugins, which has a non-trivial barrier. For research groups wanting to do RL fine-tuning on their own agent systems, this may be the most complete open-source option available.

