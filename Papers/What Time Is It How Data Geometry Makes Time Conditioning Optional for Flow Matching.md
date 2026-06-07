---
state: skimmed
name: What Time Is It? How Data Geometry Makes Time Conditioning Optional for Flow Matching
link: https://arxiv.org/abs/2605.08344v1
tldr: Showed that flow models without time conditioning have a decomposed loss and the model learns to account for time in the subspace spanned by the data (noise -> high dim; data -> low dim)
note:
quality:
  - banger
abstract: "Recent work has shown that models flow matching models can be trained without explicit time conditioning, challenging the standard view that the interpolation time is needed to disambiguate velocity targets. But why should a time-blind model work at all? Decomposing the time-blind flow matching loss, we identify two sources of irreducible error: a coupling variance, which arises from ambiguous velocity targets induced by how noise and data points are paired, and the time-blindness gap, which is the additional error caused by ignoring time. This gap shows that time-blind training is strictly harder than conventional training, reinforcing the puzzle that time-blind models work so well in practice. We resolve this tension by showing that the geometry of high-dimensional data makes time identifiable directly from noisy observations. When data concentrates near a $k$-dimensional subspace, time can be recovered from the statistical structure of noisy interpolants in directions orthogonal to the data; under a spiked-covariance model, this yields a closed-form estimator that recovers $t$ from a single observation $z$ at rate $O(1/\\sqrt{d-k})$ for ambient dimension $d$. As a consequence, we prove that the time-blindness gap is asymptotically negligible relative to the coupling variance. We empirically demonstrate our identifiability result on real-world data and show that changing the coupling has a much larger effect on loss and sample quality than removing time conditioning across CIFAR-10, CelebA-HQ, and FFHQ. These results explain why time-blind flow matching works and show that the main practical lever is the choice of coupling, not explicit time conditioning."
paper id: 2605.08344v1
authors:
  - Alec Helbling
  - Sebastian Gutierrez Hernandez
  - Benjamin Hoover
  - Duen Horng Chau
  - Parikshit Ram
publication date: 2026-05-08T18:00
comments: ""
pdf: https://arxiv.org/pdf/2605.08344v1
tags:
  - flow
  - xai
  - cv
  - talking-masks
---
#paper
## Takeaways
-

## I+D
-

## Deep Dive

