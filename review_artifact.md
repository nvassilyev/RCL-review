# Research Artifact: Reflective Context Learning (RCL)

This document hosts supplementary results referenced in our rebuttal: the
updated main-results table (now including the GEPA baseline), the
leave-one-out ablation of the composed optimizer, and a per-primitive
computational-overhead breakdown. Raw reflection/mutation traces, learned
playbooks, prompts, and reproduction configs are available elsewhere in this
repository.

---

## 1. Main Results with GEPA Baseline (Table 3)

![Table 3 — Main results with the GEPA baseline added. Each primitive is added individually to the ACE baseline; the bottom row is the full composed RCL optimizer. AppWorld: TGC; BrowseComp+/RewardBench2: accuracy. Lite = Gemini 3.1 Flash-Lite, Nano = GPT-5.4 Nano. All runs use Claude Opus as reflector/mutator with 30 training iterations. Deltas vs. Seed.](assets/table3.png)

We add **GEPA** (Agrawal et al., 2025) as a second baseline, using the official
DSPy implementation with `auto="light"` (the setting that most closely matches
our >30-iteration budget), batch size $B{=}3$, and per-benchmark validation sets
(56 tasks for AppWorld, 30 queries for BrowseComp+, 277 examples for
RewardBench2). GEPA is competitive on BrowseComp+ and RewardBench2 but unstable
on AppWorld, dropping below Seed on three of four splits; RCL never drops below
Seed in any of the eight conditions.

---

## 2. Leave-One-Out Ablation of the Composed Optimizer (Table 4)

![Table 4 — Leave-one-out ablation of the composed RCL optimizer. Each row removes one primitive; deltas are relative to full RCL. Negative = the primitive contributes positively in composition; positive = removing it helps. Largest degradation per column is underlined.](assets/table4.png)

Table 3 measures the marginal value of *adding* a primitive to ACE; Table 4
measures its role once the full optimizer is assembled. These differ. Key points:

- **Standalone value does not predict compositional role.** Auxiliary losses
  help 7/8 settings standalone, yet their removal barely hurts and even improves
  RewardBench2/Nano (+12.6). Credit assignment is the converse — it helps only
  3/8 standalone, but its removal causes the largest drop in three settings.
- **Two primitives are consistently load-bearing.** Removing grouped rollouts
  hurts 7/8 settings (largest: −9.5 AppWorld Normal/Lite, −11.3 RewardBench2/Nano);
  removing failure replay yields the single largest regression in the table
  (−18.0 BrowseComp+/Nano, −8.8 AppWorld Challenge/Nano).
- **Interactions are regime-dependent.** BrowseComp+/Nano is most sensitive to
  removing replay, batching, and credit assignment; on RewardBench2/Nano,
  removing auxiliary losses (+12.6) and credit assignment (+5.3) instead helps,
  consistent with over-structuring hurting Nano on this near-deterministic task.

Contributions are real but non-additive: effects depend on the task regime, the
agent, and the other mechanisms combined.

---

## 3. Computational Overhead

We give the per-primitive cost relative to ACE below. Cost is dominated by two
quantities: extra agent rollouts (execution) and extra optimizer/Opus calls;
all learned artifacts are text (playbook, state document, replay buffer), so
the memory footprint is negligible and constant in model size.

| Primitive | Extra agent rollouts | Extra optimizer calls | Memory |
|---|---|---|---|
| ACE (reference) | — | — | — |
| Batching ($B{=}3$) | $\sim B\times$ per iteration | up to $B\times$ reflection calls | — |
| Grouped rollouts ($G{=}3$) | $G\times$ per task | — | — |
| Credit assignment | $+1$ per task (annotated run) | larger reflector input | — |
| Auxiliary losses | none | none | — |
| Failure replay | none | none | buffer of task IDs/metadata |
| Optimizer state | none | $+1$ Opus call per iteration | rolling state document |

Two of the cheapest primitives — optimizer state and auxiliary losses — are
nonetheless among the most effective, and add **no extra agent rollouts**: auxiliary losses only restructure the reflection
prompt, and optimizer state adds a single state-update call per *iteration*
(independent of $B$), which is the largest non-execution cost but constant
across the run. The execution-side primitives (grouped rollouts, credit
assignment, batching) carry real rollout cost that scales with $G$, $B$, or
task count. This breakdown reinforces our central finding that diagnostic
precision, not execution volume, is the binding constraint. We will add this
table to the appendix and note the execution-side and optimizer-state costs in
a Limitations section.