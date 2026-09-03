---
state: skimmed
name: "ConceptAttention: Diffusion Transformers Learn Highly Interpretable Features"
link: https://arxiv.org/abs/2502.04320v2
tldr: Used self-attention to produce saliency maps of ViTs by just measuring the attention w.r.t. the embedding of concepts
note:
quality:
  - banger
abstract: Do the rich representations of multi-modal diffusion transformers (DiTs) exhibit unique properties that enhance their interpretability? We introduce ConceptAttention, a novel method that leverages the expressive power of DiT attention layers to generate high-quality saliency maps that precisely locate textual concepts within images. Without requiring additional training, ConceptAttention repurposes the parameters of DiT attention layers to produce highly contextualized concept embeddings, contributing the major discovery that performing linear projections in the output space of DiT attention layers yields significantly sharper saliency maps compared to commonly used cross-attention maps. ConceptAttention even achieves state-of-the-art performance on zero-shot image segmentation benchmarks, outperforming 15 other zero-shot interpretability methods on the ImageNet-Segmentation dataset. ConceptAttention works for popular image models and even seamlessly generalizes to video generation. Our work contributes the first evidence that the representations of multi-modal DiTs are highly transferable to vision tasks like segmentation.
paper id: 2502.04320v2
authors:
  - Alec Helbling
  - Tuna Han Salih Meral
  - Ben Hoover
  - Pinar Yanardag
  - Duen Horng Chau
publication date: 2025-02-06T18:59
comments: Oral Presentation at ICML 2025, Best Paper Award at CVPR Workshop on Visual Concepts
pdf: https://arxiv.org/pdf/2502.04320v2
tags:
  - xai
  - sde
  - cv
---
#paper
## Takeaways
- Saliency maps produce a heatmap over which we visualise activating features of an image. Most of the times we use gradient-based attribution, which takes a gradient and propagates it back. ConceptAttention is much simpler and works pretty well.
- The idea is to have a set of concepts (textual) that we want to get the saliency of, so we encode "dog", "cat", "frog", etc as a sequence of normal text tokens. Then we treat these tokens as added to the DiT (because this works only for transformer-based models) but we do a sort of cross-attention in which the patch tokens cannot attend to these concept tokens, but the concept tokens can attend to them and themselves. So, the concept tokens get updated (because they start from the very first layer) with that cross-attention and this automatically produces the saliency maps we wanted to: 
1. Get the QKV of the concept tokens by passing them through the QKV matrices of the prompt (as in multi-modal DiTs patches and prompt have different QKVs; r is the number of concepts):
$$k _ { c } = [ K _ { p } c _ { 1 }, \dots ], q _ { c } = [ Q _ { p } c _ { 1 }, \dots ], v _ { c } = [ V _ { p } c _ { 1 }, \dots ] \in \mathbb { R } ^ { r \times d }$$
2. Make K and V sequences so that we can compute the cross-attention of concepts:
$$k _ { x c } = [ K _ { x } x _ { 1 } \dots, K _ { x } x _ { n }, K _ { p } c _ { 1 } \dots, K _ { p } c _ { r } ], \, \, v _ { x c } = [ V _ { x } x _ { 1 } \dots, V _ { x } x _ { n }, V _ { p } c _ { 1 } \dots, V _ { p } c _ { r } ]$$
3. Compute the cross-attention and write-back the outputs:
$$o _ { c } = \mathrm { s o f t m a x } ( q _ { c } k _ { x c } ^ { T } ) v _ { x c }$$
4. Once we have the output from self-attention (thus have contextualised the concepts), we take a simple dot-product similarity of the concepts outputs' and the patches ones (this gives us the saliency map we're looking for): $$\phi ( o _ { x }, o _ { c } ) = \mathrm { s o f t m a x } ( o _ { x } o _ { c } ^ { T } )$$
This gives us one saliency map per concept (as we do the dot-product with all the patches, getting n similarities that we can rearrange in 2D to get a saliency map).
![[Pasted image 20260903161246.png]]
## I+D
- The same concept of embedding concepts and measuring similarity can be done for other more abstract concepts, this will give us an interpretation of how similarly this abstract concept relates to the rest of the sequence (e.g. we can analyse the self-attention that a probe encoding happiness relates to). With this we may be able to develop a technique of self-interepretation in which the model's q, k, v or o's serve as a way to measure concept similarity

## Deep Dive

