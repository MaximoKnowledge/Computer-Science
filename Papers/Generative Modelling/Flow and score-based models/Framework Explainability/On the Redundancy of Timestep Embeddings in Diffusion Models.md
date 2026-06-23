---
state: skimmed
name: On the Redundancy of Timestep Embeddings in Diffusion Models
link: https://arxiv.org/abs/2606.20416v1
tldr: Proved that one can train time-embedding free diffusion models
note:
quality:
  - good
abstract: Diffusion models rely heavily on explicit timestep embeddings to modulate the denoising process across various noise scales. In this work, we challenge the necessity of these temporal signals by analyzing their impact on U-Net and Diffusion Transformer architectures. Beyond empirical evidence, we provide a theoretical framework demonstrating that, under certain conditions, the global minimizer of the diffusion training objective can be achieved without explicit timestep conditioning. Our findings reveal a surprising robustness when timestep embeddings are completely removed. Extensive ablation studies on the CelebA and CIFAR-10 datasets show that these time-agnostic models can maintain high structural fidelity and even surpass their conditioned counterparts in competitive metrics, including FID, precision, and recall. Our analysis suggests these architectures can implicitly infer noise scales from the corrupted input under specific assumptions, rendering explicit temporal conditioning redundant. This study challenges long-standing temporal conditioning paradigms and paves the way for more efficient and structurally focused generative architectures.
paper id: 2606.20416v1
authors:
  - José A. Chávez
publication date: 2026-06-18T16:01
comments: 17 pages
pdf: https://arxiv.org/pdf/2606.20416v1
tags:
  - generative
  - architecture
  - xai
---
#paper
## Takeaways
- Proves that one can define a time-agnostic DDPM model that still converges to the same model as the time-conditioned one.  Relies on the existence of a measurable map that he later proves that it exists on probability convergence (i.e. it's not immediately available but on convergence of dimensions it exists)
- Trains a conditional and unconditional model and shows that they perform similarly in Celeba and CIFAR-10
- He shows that on natural images (under certain acceptable assumptions) the map can be recovered from the norm of $x_{t}$

## I+D
- He doesn't quite show how fast convergence for the map is, thus it remains open how many dimensions are needed to actually converge.
- Moreover although the model theoretically converges to the time-conditional one optimisation gaps may prevent it from doing so. A toy example would show how the two are equivalent
- It would be good to expand to few more datasets, show some toy examples proving the claims of convergence, and maybe causally prove what happens to the unconditional network if you change the norm (which would show if it's actually using the norm to track time)

## Deep Dive

