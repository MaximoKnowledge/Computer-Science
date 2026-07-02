---
state: read
name: "DLM-Scope: Mechanistic Interpretability of Diffusion Language Models via Sparse Autoencoders"
link: https://arxiv.org/abs/2602.05859v1
tldr: Trained first SAE on DLMs and steered models according to automatic feature detection
note:
quality:
  - good
abstract: "Sparse autoencoders (SAEs) have become a standard tool for mechanistic interpretability in autoregressive large language models (LLMs), enabling researchers to extract sparse, human-interpretable features and intervene on model behavior. Recently, as diffusion language models (DLMs) have become an increasingly promising alternative to the autoregressive LLMs, it is essential to develop tailored mechanistic interpretability tools for this emerging class of models. In this work, we present DLM-Scope, the first SAE-based interpretability framework for DLMs, and demonstrate that trained Top-K SAEs can faithfully extract interpretable features. Notably, we find that inserting SAEs affects DLMs differently than autoregressive LLMs: while SAE insertion in LLMs typically incurs a loss penalty, in DLMs it can reduce cross-entropy loss when applied to early layers, a phenomenon absent or markedly weaker in LLMs. Additionally, SAE features in DLMs enable more effective diffusion-time interventions, often outperforming LLM steering. Moreover, we pioneer certain new SAE-based research directions for DLMs: we show that SAEs can provide useful signals for DLM decoding order; and the SAE features are stable during the post-training phase of DLMs. Our work establishes a foundation for mechanistic interpretability in DLMs and shows a great potential of applying SAEs to DLM-related tasks and algorithms."
paper id: 2602.05859v1
authors:
  - Xu Wang
  - Bingqing Jiang
  - Yu Wan
  - Baosong Yang
  - Lingpeng Kong
  - Difan Zou
publication date: 2026-02-05T16:41
comments: 23 pages
pdf: https://arxiv.org/pdf/2602.05859v1
tags:
  - dlms
  - xai
  - talking-masks
---
#paper
## Takeaways
* Trained Top-K SAEs on DLM residual streams for both MASK and UNMASK token positions
  (separate SAEs per strategy). Width 16K, 6 layers × 6 sparsity levels per model.
* Inserting SAEs into early DLM layers can *reduce* cross-entropy loss (negative ΔLM loss),
  unlike autoregressive LLMs where insertion always hurts. Effect disappears in deeper layers.
* Auto-interpretation: for each feature, show a judge LLM the top-activating contexts with
  highlighted tokens → it generates a one-sentence explanation → a separate pass gives the
  judge unlabeled sequences + that explanation and asks it to predict which activate →
  accuracy = interpretability score. Confirms DLM-SAE features are human-interpretable.
* Steering: inject a feature's decoder direction into the residual stream at every denoising
  step (not just once like in AR LLMs). Two position-selection strategies per step:
  ALL-TOKENS (inject everywhere) vs UPDATE-TOKENS (inject only at currently masked positions).
  Dream worked better with all-tokens, LLaDA with update-tokens.
  DLM steering outperforms LLM steering by 2–10× in deep-layer steering scores.
* Measure activated-feature overlap (Jaccard on top-K features) across timesteps under
  different unmasking policies. Confidence-based orders (entropy, margin) show high early
  feature turnover on masked tokens then stabilize; random order stays flat throughout.
  Decoded tokens keep drifting in deep layers under confidence-based orders (bidirectional
  attention effect). This correlates with task performance (entropy/margin >> random on GSM8K).
* Base-trained SAEs transfer to SFT models nearly losslessly at all layers except the deepest
  (L27), where the base SAE can't capture instruction-tuning-specific directions.

## I+D
-

## Deep Dive

