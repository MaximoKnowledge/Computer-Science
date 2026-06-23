---
state: read
name: "Diamond Maps: Efficient Reward Alignment via Stochastic Flow Maps"
link: https://arxiv.org/abs/2602.05993v3
tldr: Proposed Stochastic Flow Maps to get Value function estimations from noisy states (similar to Meta Flow Maps)
note:
quality:
  - good
abstract: Flow and diffusion models produce high-quality samples, but adapting them to user preferences or constraints post-training remains costly and brittle, a challenge commonly called reward alignment. We argue that efficient reward alignment should be a property of the generative model itself, not an afterthought, and redesign the model for adaptability. We propose "Diamond Maps", stochastic flow map models that enable efficient and accurate alignment to arbitrary rewards at inference time. Diamond Maps amortize many simulation steps into a single-step sampler, like flow maps, while preserving the stochasticity required for optimal reward alignment. This design makes search, Sequential Monte Carlo, and guidance scalable by enabling efficient and consistent estimation of the value function. Our experiments show that Diamond Maps can be learned efficiently via distillation from GLASS Flows, achieve stronger reward alignment performance, and scale better than existing methods. Our results point toward a practical route to generative models that can be rapidly adapted to arbitrary preferences and constraints at inference time.
paper id: 2602.05993v3
authors:
  - Peter Holderrieth
  - Douglas Chen
  - Luca Eyring
  - Ishin Shah
  - Giri Anantharaman
  - Yutong He
  - Zeynep Akata
  - Tommi Jaakkola
  - Nicholas Matthew Boffi
  - Max Simchowitz
publication date: 2026-02-05T18:42
comments: ""
pdf: https://arxiv.org/pdf/2602.05993v3
tags:
  - generative
  - sde
  - rl
---
#paper
## Takeaways
- Proposed Posterior Diamond Maps and Weighted Diamond Maps. Concept is always the same: take some noisy samples, have a good one-step model to see where it lands, and use reward function at the end to approximate the value function on expectation
- The proposed models are different from the one being steered. They train independent Posterior and Weighted Diamond Maps to work as one-step approximators of image latents (i.e. given an intermediate denoising sample to denoise it in 1 step) so as to apply the reward function cleanly
- The difference in the definition of Posterior vs Weighted Maps is that the Posterior Maps is trained by distilling a GLASS Flow model (another type of model that's also similar to flow maps) and making a model that takes as context (as in meta flow-maps) the noisy latent and the time, and based on a random noise sample it tries to amortise the distribution of $p(\cdot \mid x_t, t)$, so the function looks like $X_{0,1}(\epsilon \mid x_t,t)$. The Weighted Map is similar in spirit but instead of training a model on this context, it instead does a simple trick of renoising and doing a one-step denoising. So you take a sample at time $t$ put back some Gaussian noise to go back to $t'$ and do a one-step denoising of the obtained latent. Note that the Gaussian noises are independent, so while we always go back to $t'$ the $x_{t'}$ are different among themselves. This also has some implications for the value function that can be solved mathematically
- Weighted Maps are more straightforward as yo can use a flow map already available, while the Posterior requires the intermediate GLASS model training. Both are slow at inference
- Weighted Maps can also be used to estimate the value function directly (so not just the gradient, because for steering we always use the gradient, but maybe we want the value at a point)

## I+D
- It's extremely slow and introduces a computational overhead that the gains don't justify that much. Smaller models should be studied

## Deep Dive

