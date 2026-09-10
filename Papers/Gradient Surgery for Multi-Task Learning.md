---
state: skimmed
name: Gradient Surgery for Multi-Task Learning
link: https://arxiv.org/abs/2001.06782v4
tldr: Proposed a technique to align gradients for better optimisation dynamics on multi-task learning
note:
quality:
  - banger
abstract: While deep learning and deep reinforcement learning (RL) systems have demonstrated impressive results in domains such as image classification, game playing, and robotic control, data efficiency remains a major challenge. Multi-task learning has emerged as a promising approach for sharing structure across multiple tasks to enable more efficient learning. However, the multi-task setting presents a number of optimization challenges, making it difficult to realize large efficiency gains compared to learning tasks independently. The reasons why multi-task learning is so challenging compared to single-task learning are not fully understood. In this work, we identify a set of three conditions of the multi-task optimization landscape that cause detrimental gradient interference, and develop a simple yet general approach for avoiding such interference between task gradients. We propose a form of gradient surgery that projects a task's gradient onto the normal plane of the gradient of any other task that has a conflicting gradient. On a series of challenging multi-task supervised and multi-task RL problems, this approach leads to substantial gains in efficiency and performance. Further, it is model-agnostic and can be combined with previously-proposed multi-task architectures for enhanced performance.
paper id: 2001.06782v4
authors:
  - Tianhe Yu
  - Saurabh Kumar
  - Abhishek Gupta
  - Sergey Levine
  - Karol Hausman
  - Chelsea Finn
publication date: 2020-01-19T06:33
comments: NeurIPS 2020. Code is available at https://github.com/tianheyu927/PCGrad
pdf: https://arxiv.org/pdf/2001.06782v4
tags:
  - rl
  - theory
  - twisted-maps
---
#paper
## Takeaways
- When doing multi-task learning sometimes the gradients of the losses are not aligned. The setup is this:
$$\mathcal { L } ( \theta ) = \sum _ { i } \mathcal { L } _ { i } ( \theta )$$
And the gradients w.r.t. some of the individual losses are not aligned: 
$$\mathbf { g } _ { i } = \nabla \mathcal { L } _ { i } ( \theta )$$
So: $\cos(g_{i}, g_{j})<0$
This is normal (as the gradient of the multi-task loss is just the average of the individual gradients), but if the gradients have substantially-different norms one will dominate the other, which ends up being detrimental for learning the underrepresented loss and thus the overall task is not fulfilled.
- Proposed metrics to diagnose gradient-alignment in multi-task settings:
	- **Gradient magnitude similarity**: $\Phi ( { \bf g } _ { i }, { \bf g } _ { j } ) = \frac { 2 \| { \bf g } _ { i } \| _ { 2 } \| { \bf g } _ { j } \| _ { 2 } } { \| { \bf g } _ { i } \| _ { 2 } ^ { 2 } + \| { \bf g } _ { j } \| _ { 2 } ^ { 2 } }$. Measures how much gradient i and j have a similar norm (1 is that they have = norm, ~0 that they differ substantially so one is orders of magnitude bigger than the other)
	- **Gradient similarity**: $\cos(g_{i}, g_{j})$. Just the regular cosine similarity
	- A curvature metric that is not fully understandable
- The whole method is just based on orthogonalising the gradients:
$$\mathbf { g } _ { i } = \mathbf { g } _ { i } - \frac { \mathbf { g } _ { i } { \cdot } \mathbf { g } _ { j } } { \| \mathbf { g } _ { j } \| ^ { 2 } } \mathbf { g } _ { j }$$
So, we project the gradient of i into j and subtract that projection. They propose an iterative algorithm that does that for all conflicting gradients and prove that it converges to the same solution as a normal gradient update (under some Lipschitz and learning rate assumptions)
## I+D
-

## Deep Dive

