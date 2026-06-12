## Core Idea

We consider a frozen unconditional image generator, such as a pretrained diffusion model, and learn only a small inference-time control network that steers the sampling dynamics toward user-specified objectives.

The base generative model remains fixed. The learned component is a lightweight controller that modifies the sampling trajectory without retraining the full generator.

We use the interpolant / flow-matching convention throughout: $t \in [0, 1]$, time runs forward, $X_0 \sim$ prior (noise), and $X_1$ is the clean generated sample. Rewards are read at the clean endpoint $X_1$.

The proposed controlled dynamics are:

$$
dX_t = \left[ b_\theta(t, X_t) + \sigma(t)\, u_\phi(t, X_t, e_R) \right] dt + \sigma(t)\, dW_t.
$$

Here:

- $b_\theta(t, X_t)$ is the drift induced by the frozen pretrained generator.
- $\sigma(t)$ is the diffusion coefficient.
- $u_\phi(t, X_t, e_R)$ is the learned controller.
- $e_R$ is an embedding of the reward function or objective.
- Only $\phi$, and possibly the reward encoder parameters, are trained.

The objective is not to train a new generator, but to learn a reusable steering mechanism for a fixed generator.

## Motivation

Existing reward-guided or preference-guided generative methods often require one of the following:

- full or partial fine-tuning of the generative model,
- reward-specific guidance at inference time,
- repeated gradient computations through a reward model,
- value-function estimation,
- costly trajectory-level optimization.

The proposed approach instead learns a small controller that can be plugged into the sampler at inference time. This creates a modular system:

$$
\text{frozen generator} + \text{reward-conditioned control adapter}.
$$

This is useful because the original model is preserved, while different objectives can be handled by changing the reward context given to the controller.

## Reward-Conditioned Control

Rather than learning a separate controller for each reward, the controller is conditioned on a representation of the reward function.

A reward function $R$ can be represented through a small context set:

$$
\mathcal{C}_R = \{(y_i, R(y_i))\}_{i=1}^K.
$$

Each $y_i$ is an image, latent, or generated sample, and $R(y_i)$ is the score assigned by the reward function.

A reward encoder maps this context into an embedding:

$$
e_R = A_\eta(\mathcal{C}_R).
$$

Then the controller uses this reward embedding:

$$
u_\phi(t, x_t, e_R).
$$

This allows the same controller to adapt to different objectives.

## Training Setup

The training objective below is standard stochastic optimal control; we adopt it as foundation rather than claim it as a contribution.

We define a family of training reward functions:

$$
\mathcal{R}_{\text{train}} = \{R_1, R_2, \dots, R_N\}.
$$

For each reward $R_j$, we construct a small context set:

$$
\mathcal{C}_{R_j} = \{(y_i, R_j(y_i))\}_{i=1}^K.
$$

The encoder produces:

$$
e_{R_j} = A_\eta(\mathcal{C}_{R_j}).
$$

The controlled sampler is then run using:

$$
dX_t = \left[ b_\theta(t, X_t) + \sigma(t)\, u_\phi(t, X_t, e_{R_j}) \right] dt + \sigma(t)\, dW_t.
$$

The controller is trained to maximize the final reward while staying close to the original generator:

$$
\max_{\phi, \eta} \; \mathbb{E}\!\left[ R_j(X_1) - \frac{\lambda}{2} \int_0^1 \left\| u_\phi(t, X_t, e_{R_j}) \right\|^2 dt \right].
$$

The first term encourages high-reward samples. The second term penalizes excessive deviation from the frozen base model; via Girsanov it is a path-space KL to the base process.

We take as primary the soft-value parameterization, in which the controller is the gradient of a reward-conditioned value function:

$$
V^R_t(x) = \log \mathbb{E}\!\left[ e^{R(X_1)/\lambda} \mid X_t = x \right], \qquad u^\star_R(t, x) \propto \sigma(t)\, \nabla_x V^R_t(x).
$$

This is the control-theoretically natural object: the optimal drift is the gradient of the soft value, equivalently the Doob $h$-transform that tilts the base toward $p_R(x) \propto p_{\text{base}}(x)\, e^{R(x)/\lambda}$. A free-form controller $u_\phi$ is kept as a fallback, trained from the same terminal signal.

Training needs a learning signal tied to $R(X_1)$, but integrating the controlled SDE from an intermediate state $X_t$ all the way to $X_1$ is the expensive inner loop. We avoid the full rollout using stochastic flow maps. A flow map collapses many integration steps into a single network evaluation; in the stochastic form of Meta Flow Maps (arXiv 2601.14430) and Diamond Maps (arXiv 2602.05993), it samples the posterior $p_{1|t}(x_1 \mid x_t)$ — the clean endpoints consistent with a noisy $x_t$ — in one step. At each visited $(t, X_t)$ we draw one-step posterior samples $\hat{x}_1 \sim p_{1|t}(\cdot \mid X_t)$ and evaluate $R(\hat{x}_1)$, giving a cheap, differentiable estimate of $V^R_t$ and $\nabla_x V^R_t$ at non-terminal states without simulating to $t = 1$. The training algorithm is then value-matching against these one-step posterior samples.

## Testing Generalization

To evaluate whether the controller has learned a general steering mechanism, we test on held-out reward functions:

$$
\mathcal{R}_{\text{test}} \cap \mathcal{R}_{\text{train}} = \emptyset.
$$

At test time, the model receives only a small context set:

$$
\mathcal{C}_{R_{\text{test}}} = \{(y_i, R_{\text{test}}(y_i))\}_{i=1}^K.
$$

The reward encoder produces $e_{R_{\text{test}}}$, and the same controller is used to steer the frozen generator.

The central experimental question is:

> Can a small reward-conditioned controller steer a frozen image generator toward unseen reward functions?

## Amortized Control and Meta-Generation

The setup is best understood as amortization, and it helps to separate two kinds.

The first is amortization of integration steps: this is what flow maps already provide, collapsing many simulation steps into a single network evaluation. It is the engine used in training and is not our contribution.

The second is amortization across reward functions, which is the contribution. Each reward $R$ induces a distinct stochastic control problem whose solution is the optimal drift $u^\star_R \propto \sigma \nabla_x V^R$ — the Doob $h$-transform tilting the base toward $p_R(x) \propto p_{\text{base}}(x)\, e^{R(x)/\lambda}$. The standard regime solves one such problem per reward by fine-tuning, re-guiding, or re-estimating the value. We instead learn a single conditional map

$$
R \longmapsto u^\star_R, \qquad \text{realized as } u_\phi(t, x, e_R), \quad e_R = A_\eta(\mathcal{C}_R),
$$

so that one network emits the controlled process for any reward in the family from a few evaluations. This is the solution operator of a reward-indexed family of control problems: an in-context map from a reward (a function, summarized by $\mathcal{C}_R$) to its optimal control (a function). Held-out rewards test whether this operator was learned or whether $N$ point solutions were memorized.

The two amortizations compose: the step-amortized flow maps supply the cheap value targets at non-terminal states that make training the reward-amortized controller tractable.

The practical payoff is the central selling point. After a one-time training cost, adapting to a new objective costs only $K$ clean reward evaluations to form $\mathcal{C}_R$, and then no reward queries, no reward gradients, and no optimization at sampling time; the control is produced in a single forward pass of the encoder.

## Possible Reward Families

To avoid the complexity of text-to-image generation, the first experiments can use image-only rewards.

Possible rewards include:

- brightness,
- contrast,
- symmetry,
- edge density,
- color histogram matching,
- distance to a target latent embedding,
- classifier confidence,
- object category score,
- stroke thickness,
- center of mass,
- shape compactness.

A simple parametric reward family could be:

$$
R_\omega(x) = -\|F(x) - \omega\|^2,
$$

where $F(x)$ extracts image features and $\omega$ defines the desired target feature vector.

Training can use many values of $\omega$, while testing uses held-out values or held-out feature combinations.

## Relationship to Classifier Guidance

Classifier guidance steers generation using the gradient of a fixed classifier or condition. The present method generalizes this: rather than a single fixed objective, the controller is conditioned on a representation of the reward function itself,

$$
u_\phi(t, x, e_R),
$$

and is trained to infer how to steer from a small set of reward evaluations. Fixed-classifier guidance is recovered as the special case of a single, known objective; the added ingredient is generalization across reward functions.

## Relationship to Fine-Tuning

The base generator remains frozen ($\theta$ fixed); only the controller and reward encoder are trained ($\phi, \eta$). This places the method on the adapter side of the spectrum rather than full fine-tuning, with the usual adapter benefits: fewer trainable parameters, modularity, preservation of the original generator, multiple objective-specific behaviors from one base, and reduced training cost. Unlike a per-objective adapter, a single controller is meant to cover a family of objectives.

## Main Claim

The main claim should be stated carefully:

> We learn a lightweight reward-conditioned control adapter that enables a frozen image generator to be steered toward unseen image-level rewards from a small context of reward evaluations.

The claim should not be:

> The method works for any reward function.

A more realistic and defensible claim is:

> The method generalizes to unseen reward functions drawn from a structured reward family.

## Key Contributions

The novel contributions are the reward encoder and the reward-generalizing (held-out) evaluation. The controlled-SDE formulation is adopted from stochastic optimal control, and the rollout-free training signal is adopted from recent few-step posterior-sampling methods.

1. A reward encoder $A_\eta$ mapping a small context set of reward evaluations into an objective embedding $e_R$.
2. A reward-conditioned controller $u_\phi(t, x, e_R)$ — equivalently a reward-conditioned value $V(t, x, e_R)$ — trained across a family of reward functions and evaluated on held-out rewards, amortizing the family of control problems into a single map.
3. An image-only experimental framework that avoids the additional complexity of text-to-image generation.

## Evaluation

The method can be evaluated using:

- reward improvement over the frozen base model,
- reward-vs-quality Pareto curves over a $\lambda$ sweep, rather than single point numbers,
- sample quality and diversity metrics,
- control energy and inference overhead,
- generalization to held-out reward parameters and to held-out reward types,
- leakage diagnostics — shuffled context labels and swapped reward embeddings — to confirm the controller uses $e_R$ rather than applying one generic push,
- comparison against fixed guidance, reward-gradient guidance, and small fine-tuning baselines.

## Research Framing

This project sits at the intersection of stochastic optimal control, diffusion-model steering, reward-conditioned generation, adapter-based model modification, meta-learning over objectives, and few-shot adaptation.

The framing is to treat reward alignment not as full fine-tuning and not as fixed classifier guidance, but as amortized stochastic optimal control over a reward-indexed family of tilted target distributions:

$$
\text{frozen generator} + \text{objective-conditioned controller}.
$$

## Short Summary

We freeze a pretrained image generator and learn a small controller that steers its sampling dynamics. The controller is conditioned on an embedding of a reward function, obtained from a small set of reward evaluations. During training the controller sees many reward functions; at test time it must steer the generator toward held-out rewards. This tests whether a lightweight control adapter can learn a general, amortized mechanism for reward-conditioned image generation without retraining the base model. Training avoids expensive rollouts at non-terminal states by using stochastic flow maps (Meta Flow Maps, Diamond Maps) for one-step posterior sampling, which give value estimates at non-terminal states directly.