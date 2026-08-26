---
state: skimmed
name: Mechanistic understanding and validation of large AI models with SemanticLens
link: https://arxiv.org/abs/2501.05398v1
tldr: Pipeline for semantically-analysing vision models neurons using foundational vision models
note:
quality:
  - good
abstract: Unlike human-engineered systems such as aeroplanes, where each component's role and dependencies are well understood, the inner workings of AI models remain largely opaque, hindering verifiability and undermining trust. This paper introduces SemanticLens, a universal explanation method for neural networks that maps hidden knowledge encoded by components (e.g., individual neurons) into the semantically structured, multimodal space of a foundation model such as CLIP. In this space, unique operations become possible, including (i) textual search to identify neurons encoding specific concepts, (ii) systematic analysis and comparison of model representations, (iii) automated labelling of neurons and explanation of their functional roles, and (iv) audits to validate decision-making against requirements. Fully scalable and operating without human input, SemanticLens is shown to be effective for debugging and validation, summarizing model knowledge, aligning reasoning with expectations (e.g., adherence to the ABCDE-rule in melanoma classification), and detecting components tied to spurious correlations and their associated training data. By enabling component-level understanding and validation, the proposed approach helps bridge the "trust gap" between AI models and traditional engineered systems. We provide code for SemanticLens on https://github.com/jim-berend/semanticlens and a demo on https://semanticlens.hhi-research-insights.eu.
paper id: 2501.05398v1
authors:
  - Maximilian Dreyer
  - Jim Berend
  - Tobias Labarta
  - Johanna Vielhaben
  - Thomas Wiegand
  - Sebastian Lapuschkin
  - Wojciech Samek
publication date: 2025-01-09T17:47
comments: 74 pages (18 pages manuscript, 7 pages references, 49 pages appendix)
pdf: https://arxiv.org/pdf/2501.05398v1
tags:
  - xai
  - cv
  - representation
---
#paper
## Takeaways
- Creates dataset of most-firing examples for a neuron. Then they input those same examples to a foundational vision model (e.g. CLIP, DiNO, VGG, or something like that) and constructed mean vectors. Now you have semantical embeddings for a neuron's most-firing activations. This approach is substantially clean and easy to implement. They also propose a whole pipeline and cosine alignment but didn't go through it as it seems to hand-engineered

## I+D
- This method won't work naturally for denoising models as foundational models may not be suited to interpreting and producing meaningful embeddings for them. Thus, does creating a denoising foundational model buys us something? How can we say it's better than using a classical CLIP? Answer: A workaround is to feed to CLIP/DINO or whatever, not the noisy image but the clean one produced across that trajectory. Nonetheless this would cluster neurons across trajectories and may not be fully able to distinguish if neurons change roles across denoising (at leas not as proposed)

## Deep Dive

