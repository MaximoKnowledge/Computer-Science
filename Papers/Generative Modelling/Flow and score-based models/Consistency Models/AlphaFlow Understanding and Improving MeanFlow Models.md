---
state: read
name: "AlphaFlow: Understanding and Improving MeanFlow Models"
link: https://arxiv.org/abs/2510.20771v1
tldr: Dissected meanflow's loss into two components and observed the competing dynamics of its gradients. Proposed a unifying framework for consistency, shortcut and mean flow models
note:
quality:
  - good
abstract: "MeanFlow has recently emerged as a powerful framework for few-step generative modeling trained from scratch, but its success is not yet fully understood. In this work, we show that the MeanFlow objective naturally decomposes into two parts: trajectory flow matching and trajectory consistency. Through gradient analysis, we find that these terms are strongly negatively correlated, causing optimization conflict and slow convergence. Motivated by these insights, we introduce $α$-Flow, a broad family of objectives that unifies trajectory flow matching, Shortcut Model, and MeanFlow under one formulation. By adopting a curriculum strategy that smoothly anneals from trajectory flow matching to MeanFlow, $α$-Flow disentangles the conflicting objectives, and achieves better convergence. When trained from scratch on class-conditional ImageNet-1K 256x256 with vanilla DiT backbones, $α$-Flow consistently outperforms MeanFlow across scales and settings. Our largest $α$-Flow-XL/2+ model achieves new state-of-the-art results using vanilla DiT backbones, with FID scores of 2.58 (1-NFE) and 2.15 (2-NFE)."
paper id: 2510.20771v1
authors:
  - Huijie Zhang
  - Aliaksandr Siarohin
  - Willi Menapace
  - Michael Vasilkovsky
  - Sergey Tulyakov
  - Qing Qu
  - Ivan Skorokhodov
publication date: 2025-10-23T17:45
comments: ""
pdf: https://arxiv.org/pdf/2510.20771v1
tags:
  - generative
  - cv
  - twisted-maps
---
#paper
## Takeaways

**Note**: In this paper, as in diffusion ones, t=0 is the clean sample and t=1 is the noisy one. So we go from 1 -> 0.
- Decomposed mean flow loss into two parts:
Original Mean Flow loss:
$$ { \mathcal { L } } _ { \text { MF } } ( \boldsymbol { \theta } ) = \underset { t, r, \boldsymbol { z } _ { t } } { \mathbb { E } } \left[ \left\| { \boldsymbol { u } } _ { \boldsymbol { \theta } } ( \boldsymbol { z } _ { t }, r, t ) - \boldsymbol { v } _ { t } + ( t - r ) \frac { { \sf d } { \boldsymbol { u } } _ { \boldsymbol { \theta } ^ { - } } ( \boldsymbol { z } _ { t }, r, t ) } { { \sf d } t } \right\| _ { 2 } ^ { 2 } \right]$$
The two parts:
$$\begin{aligned}
\mathcal { L } _ { \mathrm { MF } } ( \boldsymbol { \theta } ) &= \underbrace { \mathbb { E } _ { t, r, \boldsymbol { z } _ { t } } \left[ \| \boldsymbol { u } _ { \boldsymbol { \theta } } ( \boldsymbol { z } _ { t }, r, t ) - \boldsymbol { v } _ { t } \| _ { 2 } ^ { 2 } \right] } _ { \mathrm { Trajectory \, flow \, matching ~ \mathcal{L}_{\mathrm{TFM}}}} + \underbrace { \mathbb { E } _ { t, r, \boldsymbol { z } _ { t } } \left[ 2 \left( t - r \right) \cdot \boldsymbol { u } _ { \boldsymbol { \theta } } ^ { \top } ( \boldsymbol { z } _ { t }, r, t ) \frac { \mathrm { d } \boldsymbol { u } _ { \boldsymbol { \theta } } - ( \boldsymbol { z } _ { t }, r, t ) } { \mathrm { d } t } \right] } _\mathrm{  { Trajectory  \, Consistency \, \mathcal{L}_{TC_{c}}} } + C. \\
\end{aligned}$$
The trajectory flow matching essentially tries to match the flow-matching starting-point velocity at each prediction, while the consistency one is similar to ESD in flow maps or also the consistency model loss and tries to zero-out the total derivative

- The best finding is that TFM and TC are anti-aligned across training:
![[Pasted image 20260825185110.png| center |292]]

Here, FM' is just classical flow-matching loss (which is the same as trajectory flow matching, but setting r=t). Importantly, the gradient of flow matching oscillates w.r.t. TC, so it's not always anti-aligned and it seems to be 50% of the time pointing in the same direction. 
- Interestingly (but maybe not surprising, because that's why adding FM' helps), the flow matching loss is greatly aligned with the complete mean flow one, which explains why FM helps to optimise MF's loss overall:
![[Pasted image 20260826144147.png|365]]

The issue with the paper's claim that FM' is a surrogate of TFM is that the gradients don't seem to back that evidence:
![[Pasted image 20260826144301.png|324]]
- Proposed a new training scheme based on an annealing/curriculum quantity $\alpha$:
$${ \mathcal { L } } _ { \alpha } ( \boldsymbol { \theta } ) \triangleq \underset { t, r, z _ { t } } { \mathbb { E } } \left[ \alpha ^ { - 1 } \cdot \left\| { \boldsymbol { u } } _ { \boldsymbol { \theta } } ( z _ { t }, r, t ) - ( \alpha \cdot { \tilde { v } } _ { s, t } + ( 1 - \alpha ) \cdot { \boldsymbol { u } } _ { \boldsymbol { \theta } } - ( z _ { s }, r, s ) ) \right\| _ { 2 } ^ { 2 } \right]$$

- Importantly, they don't completely eliminate flow matching. And this table shows that they still get mixed results with the flow matching ratio (i.e. they still need a considerable portion of flow matching, FDD is FID but with DINO):
![[Pasted image 20260826162824.png|353]]
Here they show that the percentage of flow matching (the first column), on their method (the second row of each section is their method and the first is just mean flow). FIDs are really high because models are undertrained and with no CFG. The takeaway is that, unlike mean flows, they get mixed results when varying the diagonal fraction (which is not so good)
## I+D
-

## Deep Dive

