---
state: read
name: Latent Flow Transformer
link: https://arxiv.org/abs/2505.14513v1
tldr: Did flow-matching to transport representation of token hi to token hj (i<j). Employed a modified flow-matching called Flow Walking, to avoid path crossing (i.e. intermediate representations collapse)
note:
quality:
  - good
abstract: Transformers, the standard implementation for large language models (LLMs), typically consist of tens to hundreds of discrete layers. While more layers can lead to better performance, this approach has been challenged as far from efficient, especially given the superiority of continuous layers demonstrated by diffusion and flow-based models for image generation. We propose the Latent Flow Transformer (LFT), which replaces a block of layers with a single learned transport operator trained via flow matching, offering significant compression while maintaining compatibility with the original architecture. Additionally, we address the limitations of existing flow-based methods in \textit{preserving coupling} by introducing the Flow Walking (FW) algorithm. On the Pythia-410M model, LFT trained with flow matching compresses 6 of 24 layers and outperforms directly skipping 2 layers (KL Divergence of LM logits at 0.407 vs. 0.529), demonstrating the feasibility of this design. When trained with FW, LFT further distills 12 layers into one while reducing the KL to 0.736 surpassing that from skipping 3 layers (0.932), significantly narrowing the gap between autoregressive and flow-based generation paradigms.
paper id: 2505.14513v1
authors:
  - Yen-Chen Wu
  - Feng-Ting Liao
  - Meng-Hsi Chen
  - Pei-Chen Ho
  - Farhang Nabiei
  - Da-shan Shiu
publication date: 2025-05-20T15:41
comments: ""
pdf: https://arxiv.org/pdf/2505.14513v1
tags:
  - flow
  - efficiency
  - llm
---
#paper
## Takeaways
- When doing flow-matching with paired data $(x_{0},x_{1})$ and constructing OT paths (i.e. linear), there's a chance of flow-crossing, in which two different couplings get the same $x_t$ and in that case the solution just becomes the average. 
- They propose a Flow Walking model, that essentially integrates in three steps from $x_0$ to $x_1$ essentially avoiding crossings
- They show that among middle layers the crossing is low (recoupling ratio)
- The whole paper is bad in quality, and leaves a lot of question in whether flow-matching can be applied

## I+D
-

## Deep Dive

