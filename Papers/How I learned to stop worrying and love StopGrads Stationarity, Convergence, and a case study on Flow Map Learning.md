---
state: skimmed
name: "How I learned to stop worrying and love StopGrads: Stationarity, Convergence, and a case study on Flow Map Learning"
link: https://arxiv.org/abs/2609.16222v1
tldr: Proofs with functional theory where to place stopgrads inside flow maps; which disagrees with Boffi and has better results
note:
quality:
  - good
abstract: Stopgrads are widely used in training machine learning models, but stopgrads can alter the gradient, stationary points and convergence guarantees of the original objective, which can make stopgrad training theoretically ungrounded. We introduce a stopgrad regression principle, which identifies a general template for stopgrad objectives with a closed-form characterization of stationary points and their uniqueness, unifying stopgrad objectives for flow maps, reinforcement learning, and diffusion samplers. We provide theoretical grounding for optimizing stopgrad flow map objectives by showing their unique stationary point is the true flow map, and showing positive convergence results for Eulerian and Lagrangian objectives, including MeanFlow and improved MeanFlow. Remarkably, we show that under functional semi-gradient flow, the learned flow map has a closed-form expression composing the initial flow map and the true flow map. We additionally use our stopgrad regression principle to propose modified stopgrad placements for flow map objectives which reduce training memory by 2x.
paper id: 2609.16222v1
authors:
  - Max W. Shen
  - Mark Goldstein
  - Zichu Wang
  - Aahlad Puli
  - Rajesh Ranganath
publication date: 2026-09-14T18:51
comments: ""
pdf: https://arxiv.org/pdf/2609.16222v1
tags:
  - sde
  - generative
  - twisted-maps
  - theory
---
#paper
## Takeaways
- Main contribution is a general principle for analyzing where training with stopgrad can stop, which they use to establish guarantees for specific flow-map objectives. The current stopgrads of Boffi remain like unknown objectives. They derive slim-LSD and slim-ESD, which place the stopgradient so that only the direct prediction remains outside the stopgrad. Using the classical residual flow map parametrisation, the modification for LSD is to additionally detach the time-derivative term: $$
  \mathcal{L}_{\mathrm{LSD-orig}}[f]
  = \frac{1}{2}\mathbb{E}\left[
  \left\|
  f(t,u,\mathbf{x}_t)
  +(u-t)\partial_u f(t,u,\mathbf{x}_t)
  -\mathrm{SG}\left\{f\big(u,u,F(t,u,\mathbf{x}_t)\big)\right\}
  \right\|^2
  \right]
  $$$$
  \mathcal{L}_{\mathrm{LSD-slim}}[f]
  = \frac{1}{2}\mathbb{E}\left[
  \left\|
  f(t,u,\mathbf{x}_t)
  +(u-t)\mathrm{SG}\left\{\partial_u f(t,u,\mathbf{x}_t)\right\}
  -\mathrm{SG}\left\{f\big(u,u,F(t,u,\mathbf{x}_t)\big)\right\}
  \right\|^2
  \right]
  $$
  For ESD a similar modification of the stopgrad is proposed:
$$
  \mathcal{L}_{\mathrm{ESD-orig}}[f]
  = \frac{1}{2}\mathbb{E}\left[
  \left\|
  \partial_t F(t,u,\mathbf{x}_t)
  +\mathrm{SG}\left\{
  (\partial_x F)(t,u,\mathbf{x}_t)\,f(t,t,\mathbf{x}_t)
  \right\}
  \right\|^2
  \right]
  $$
$$
  \mathcal{L}_{\mathrm{ESD-slim}}[f]
  = \frac{1}{2}\mathbb{E}\left[
  \left\|
  f(t,u,\mathbf{x}_t)
  -\mathrm{SG}\left\{
  (u-t)\partial_t f(t,u,\mathbf{x}_t)
  +(\partial_x F)(t,u,\mathbf{x}_t)\,f(t,t,\mathbf{x}_t)
  \right\}
  \right\|^2
  \right]
  $$

  These stopgrad placements, combined with FM supervision, are guaranteed under the paper’s assumptions to have the true flow map as their **unique stationary point**, meaning the only map where the idealized learning updates stop. The proof has two parts: their general principle shows that updates stop exactly when the prediction equals its model-generated target; they then show that, for these objectives, the resulting conditions are precisely the equations defining the true flow map, with the built-in boundary condition$$
  F(t,t,\mathbf{x})=\mathbf{x}.
  $$
  Those equations have a unique solution. The general principle alone does not establish uniqueness. This assumes unlimited model capacity, smoothness and Lipschitz conditions, sampling that covers the full domain, and finite expectations. They also establish the true flow map as the unique stationary point for MeanFlow and improved MeanFlow.
  
- **Stopgrad changes the learning updates, not the numerical loss.** For MeanFlow, improved MeanFlow, slim-LSD and slim-ESD, the authors prove that there is no other smooth function inducing the same gradients (thus to optimise slim-LSD, you need to use slim-LSD there's no surrogate loss possible). Their proof shows that the updates violate a symmetry condition that gradients of any single smooth loss must satisfy.

- They separately prove convergence by deriving an explicit expression for how the learned map evolves during idealized continuous-time training and showing that it approaches the true flow map. For slim-LSD, slim-ESD and improved MeanFlow, this assumes **perfect FM pretraining**, meaning the model already predicts the exact instantaneous velocity before distillation begins—not that it already predicts the correct longer jumps. MeanFlow does not require that initialization. Perfect pretraining is needed for these convergence proofs, not for the unique-stationary-point results. With additional derivative bounds, they also obtain exponential convergence rates. These are not direct convergence guarantees for finite neural networks trained with stochastic updates.

- Their slim variants avoid backpropagating through model derivatives, reducing training memory by approximately 2× compared to the non-stopgrad objectives and 1.5× compared to Boffi’s original-stopgrad ones in the reported CIFAR-10 experiments.

- On CIFAR-10 their method is better than Boffi’s one for the ESD correction at every reported sampling budget, but worse for the LSD one in single-step sampling and comparable at 10/50/100 steps (using a diagonal fraction of 0.25, though which we know is far from optimal). 

- On ImageNet their method is better for the ESD correction but worse for the LSD one in reported mean one-step FID.
## I+D
-

## Deep Dive

