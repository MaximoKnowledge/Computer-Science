## Core Idea

We keep a pretrained image generator frozen and add an inference-time safety controller that steers sampling away from unsafe generations, in the setting where unsafe content is carried through reference images rather than through text. The controller does not reject final outputs and does not retrain the base model; it modifies the sampling dynamics using the gradient of a learned soft value function, biasing generation toward safe terminal images while staying close to the base distribution.

The motivating regime is reference-conditioned generation: a user supplies one or more reference images plus a benign prompt and asks for similar images. Unsafe visual semantics can pass through the reference channel even when the text alone is safe and would never produce unsafe content zero-shot.

## Research Question

> Can inference-time soft-value steering on the visual channel of a frozen generator suppress unsafe reference-conditioned generations — including adaptive attacks where text and image are each benign in isolation — without retraining the base model and without collapsing benign quality?

## Threat Model

- The conditioning $c$ includes text and one or more reference images.
- The prompt is benign in isolation; safety filters keyed on the prompt pass it.
- Harm is induced through the reference image, or emerges from the image–text combination.
- The adversary may iterate, probing with combinations that are individually safe but jointly unsafe.

The defining feature is that the harm signal does not live in the text. Methods that read the prompt/concept side therefore have nothing to key on.

## Scope and Exclusions

- Minor safety is out of scope for this mechanism by design. Content involving the sexualization of minors is handled by hard refusal and policy upstream of the generator. It is never a steering gradient, never a reward category traded against utility, never a benign-neighbor calibration target, and never appears in any bank, calibration set, or reward-versus-quality curve. The steering controller is engaged only for categories where graded, reversible suppression is the appropriate response.
- Continual / online learning is out of scope here. Accumulating discovered jailbreaks into an online memory bank and updating the controller over time is a separate project (nonstationary safety plus memory); mixing it in would make it impossible to attribute any gain to the value formulation rather than the memory updates. This document studies a fixed controller; the continual extension is noted as future work.

## High-Level Formulation

Notation and convention (consistent with the companion proposal): time runs forward $t \in [0,1]$, with $z_0$ the prior noise and $z_1$ the clean generation; $z_t$ is the diffusion state at time $t$. Let $s_\theta(z_t, c)$ be the frozen base score.

Single scalar safety reward. Instead of a helpful-minus-unsafe pair of models, we use one scalar reward learned from pairwise preferences (safe over unsafe) in the Bradley–Terry sense:
$$R_\psi(z_1, c) \in \mathbb{R}, \qquad \text{higher} = \text{safer}.$$
The reward is defined on clean generations and induces the tilted terminal target
$$p^\star(z_1 \mid c) \ \propto\ p_\text{base}(z_1 \mid c)\, \exp\!\big(R_\psi(z_1, c)/\lambda\big).$$

Soft value. The object that actually drives steering is the soft value of this reward at a noisy state:
$$V_t(z_t, c) \ =\ \log \mathbb{E}\big[\exp\!\big(R_\psi(z_1, c)/\lambda\big) \,\big|\, z_t, c\big],$$
the look-ahead expectation over the base posterior $p(z_1 \mid z_t, c)$ of clean completions.

Steering rule. The controlled score is the base score tilted by the value gradient (the Doob h-transform drift, up to the schedule-dependent coefficient):
$$\tilde s(z_t, c) \ =\ s_\theta(z_t, c) \ +\ \lambda_t\, \nabla_{z_t} V_t(z_t, c),$$
with $\lambda_t$ a timestep-dependent steering schedule. This biases sampling toward high-reward (safe) terminal images while remaining close to the base process.

## Soft-Value Estimation (the crux)

The hard subproblem is not the steering rule; it is estimating $V_t$, and its gradient, across all noise levels. We take soft-value estimation as the primary method.

- Primary route (posterior sampling). Draw clean completions from the base posterior $p(z_1 \mid z_t, c)$ and form a Monte Carlo soft value $V_t \approx \log \frac{1}{m}\sum_i \exp(R_\psi(z_1^{(i)}, c)/\lambda)$, with the gradient taken through the estimator. The completions can come from one-step stochastic posterior samplers (stochastic flow maps such as Meta Flow Maps / Diamond Maps), which give low-cost posterior draws without rolling the SDE out to $z_1$. This route tolerates a non-differentiable safety reward, since the reward is only evaluated on sampled clean images.
- Comparison route (noise-conditioned head). Learn a value head $V_\xi(z_t, c)$ directly by regression to bootstrapped targets across noise levels. Cheaper to deploy, harder to calibrate, and closer to a learned classifier-guidance head.

The two routes are the primary axis of comparison at the thesis level; the posterior-sampling route is primary because its novelty lives in the estimator and it is the version that does not reduce to reward-gradient guidance.

## Relationship to Inference-Time Guidance

Reward-gradient / classifier guidance. Steering by the gradient of the reward applied to the current latent — or to the Tweedie posterior mean $\hat z_1 = \mathbb{E}[z_1 \mid z_t, c]$, the DPS form $\nabla_{z_t} R_\psi(\hat z_1, c)$ — is reward-gradient guidance. It coincides with soft-value steering only when the posterior $p(z_1 \mid z_t, c)$ is concentrated; the two diverge when it is multimodal, by Jensen plus the posterior spread. The reference-conditioned regime is precisely the multimodal case: a benign-looking noisy latent can denoise to either a safe or an unsafe completion, so reward-at-the-mean averages across a bimodal posterior while the soft value does not. Soft value-based decoding is established as distinct from classifier guidance and DPS (SVDD, Li et al.), including its tolerance of non-differentiable rewards; our contribution is not the value formulation itself but its use on the visual safety channel and the demonstration of the value-versus-mean gap where it bites.

Text-keyed safety guidance. Training-free safety methods key on unsafe concepts in the text/prompt embedding space: SLD subtracts the noise conditioned on unsafe concepts, and SAFREE projects prompt token embeddings off a toxic-concept subspace with latent re-attention. In the reference-conditioned regime the prompt is benign, so these methods have no concept token to key on, and prompt-based safety guidance has been shown to fail once harm is not carried by explicit tokens. The delta of this project is the channel: the safety signal is read from the visual state and its clean-completion posterior, not from the prompt.

## Relationship to Model-Modification Safety

Concept-erasure and robust-unlearning methods (e.g. AdvUnlearn) modify the base weights to remove unsafe concepts. This project keeps $\theta$ frozen and intervenes only at inference, which preserves the base model unchanged, lets the safety behavior be turned off or recalibrated without a new training run, and avoids the collateral degradation on unrelated content that weight edits risk. The trade is that a frozen base can still represent unsafe content, so the entire safety burden falls on the value estimate and the steering schedule.

## Central Experiment: Adaptive Adversary

The single experiment that carries the project is the comparison of fixed versus adaptive adversaries, with the adversary specified precisely:

- Knowledge. Two settings: black-box to the controller (the adversary sees only generations), and white-box to the current risk model (the adversary has the value/reward and can optimize against its gradient). Report both.
- Budget. A fixed query budget per attack and a fixed number of reference images per attempt.
- Construction. The adversary searches over reference images and prompt phrasings that are individually benign but jointly elicit unsafe generations.
- Fixed vs adaptive. Fixed: attacks from a held-out set the controller was tuned against. Adaptive: attacks optimized against the deployed controller. The gap between the two is the headline result; a method that only holds against fixed attacks is reported as such.

## Main Claim

The claim should be stated narrowly:

> On the visual channel of a frozen generator, soft-value steering with a single scalar safety reward suppresses unsafe reference-conditioned generations more effectively than post-hoc filtering, reward-at-the-posterior-mean guidance, and prompt-keyed safety guidance, in the regime where the unsafe signal is carried by reference images.

It should not be stated as "this solves safe image generation." The project should stay honest that, absent the channel gap and the value-versus-mean gap shown empirically, it reduces to soft value-based decoding applied to a safety reward.

## Key Contributions

1. A frozen-generator, inference-time formulation of safety as soft-value steering, with the base model left unchanged.
2. A safety controller that reads the visual / clean-completion-posterior channel, targeting the reference-conditioned regime where prompt-keyed methods have nothing to key on.
3. A single Bradley–Terry scalar safety reward and the estimation of its soft value across all noise levels, with posterior sampling as the primary estimator and tolerance for non-differentiable rewards.
4. An ablation isolating soft value versus reward-at-the-posterior-mean, designed to fire exactly where the safe/unsafe posterior is bimodal.
5. A reference-conditioned adaptive-adversary evaluation, reported under both fixed and adaptive attacks and under both black-box and white-box-to-risk-model adversaries.

## Evaluation

Baselines, reduced to those that each isolate one thing:

- post-hoc safety filter — does steering beat filtering,
- reward-at-the-posterior-mean guidance with the same reward — does the look-ahead value matter (the core ablation),
- one prompt-keyed method, SLD or SAFREE — does the channel gap show,
- soft-value steering — the proposed method.

Metrics:

- unsafe generation rate and jailbreak success rate, under fixed and adaptive adversaries,
- benign false-positive / overblocking rate and degradation on retain benchmarks,
- image quality and reference fidelity on benign prompts,
- the soft-value-versus-mean gap as a function of posterior multimodality.

## Research Framing

The project sits at the intersection of stochastic optimal control (the h-transform / tilted-target view of guidance), diffusion steering, soft-value estimation, and inference-time AI safety and red-teaming for reference-conditioned generation. The conceptual move is to treat safety not as weight editing and not as prompt-side filtering, but as a value-steering problem on the visual channel of a frozen generator — with the honest caveat that the value-steering machinery is borrowed, so the contribution rests on the threat model and the empirical gaps.

## Short Summary

 We keep a pretrained image generator frozen and steer its sampling away from unsafe generations at inference time, in the reference-conditioned regime where the prompt is benign and harm is carried by reference images. A single Bradley–Terry safety reward defines a tilted terminal target, and steering follows the gradient of that reward's soft value at each noise level. We take soft-value estimation, via posterior sampling such as stochastic flow maps, as the primary method, because it is the version that does not reduce to reward-gradient guidance; at the thesis level we compare both the soft-value estimator and the cheaper reward-gradient route. The central experiment is a reference-conditioned adaptive adversary, reported under both fixed and adaptive attacks. The continual-memory extension is left as separate future work.