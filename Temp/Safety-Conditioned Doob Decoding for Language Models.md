## Core idea

The proposed method is an inference-time safety decoding scheme for autoregressive language models based on a Doob $h$-transform.

The model is treated as a stochastic process over prefixes. Harmful generation is modeled as a killed process: once the generated prefix enters a harmful set $H$, the trajectory is terminated or assigned zero probability.

The goal is not merely to penalize locally unsafe tokens. Instead, the goal is to sample from the base language model conditioned on the event that the generation never enters the harmful region.

In short:

> Sample from the original LM distribution conditioned on staying safe for the entire trajectory.

This is a stronger and cleaner formulation than token-level toxicity filtering because it reasons about future risk, not only current prefix risk.

## Mathematical formulation

Let $(s = y_{\le t})$ be the current generated prefix.

Let \(H\) be the harmful zone, defined by a classifier, policy model, moderation model, or other safety predicate.

Define the hitting time of the harmful set:
$$\tau_H = \inf \{ t : y_{\le t} \in H \}.$$


For a finite horizon $T$, define the survival harmonic function:
$$h_t(s) = \Pr_{p_{\mathrm{LM}}}(\tau_H > T \mid y_{\le t}=s).$$


This is the probability that, starting from prefix $s$, the base LM will avoid harmful prefixes until the end of generation.

The Doob-transformed next-token kernel is:
$$q_t(v \mid s)
=
p_t(v \mid s)
\mathbf{1}\{sv \notin H\}
\frac{h_{t+1}(sv)}{h_t(s)}.$$

Since $h_t(s)$ is a normalizer independent of the candidate token $v$, decoding can use the proportional form:

$$q_t(v \mid s)
\propto
p_t(v \mid s)
\mathbf{1}\{sv \notin H\}
h_{t+1}(sv).$$

Equivalently, in logit space:
$$\log q_t(v \mid s)
\propto
\log p_t(v \mid s)
+
\log \mathbf{1}\{sv \notin H\}
+
\log h_{t+1}(sv).$$

The hard safety mask removes tokens that immediately enter the harmful set. The $h$-term downweights tokens that may look locally safe but lead to unsafe continuations with high probability.

## Key distinction from ordinary classifier-guided decoding

A standard safety classifier asks:

> Is this candidate token or prefix harmful now?

The Doob version asks:

> If we choose this token, how likely is the future generation to remain safe?

Therefore, the classifier should not merely estimate instantaneous harmfulness. The central object should be a survival value:
$$h(s) = \Pr(\text{no future harmful prefix} \mid s).$$


This is the main conceptual difference from ordinary toxicity penalties, rejection sampling, or local moderation.

## Proposed method

The method can be called one of the following:

- Doob Safety Decoding
- Survival-Conditioned Decoding
- Harmonic Safety Decoding
- Safety Doob Transform
- Killed-Process Safety Decoding

At each generation step:

1. The base LM proposes next-token probabilities $p(v \mid s)$.
2. A safety classifier checks whether the candidate prefix $sv$ is already harmful.
3. A survival model estimates $\hat h(sv)$, the probability that generation from $sv$ will avoid $H$ until termination.
4. Candidate token probabilities are reweighted by:
$$\mathrm{score}(v)
=
\log p(v \mid s) + \log \hat h(sv),$$

with zero probability assigned to prefixes already classified as harmful.

The resulting decoder samples from a safety-conditioned approximation of the original LM.

## Variants

### Variant A: Rollout-based survival estimator

For each top-$K$ candidate token $v$, sample $N$ continuations from the base LM.

Run a streaming safety classifier on each continuation.

Estimate:
$$\hat h(sv)
=
\frac{1}{N}
\sum_{i=1}^{N}
\mathbf{1}\{\tau_H^{(i)} > T\}.$$


This is expensive but conceptually clean. It can serve as an oracle-style baseline.

Advantages:

- Closest to the pure Doob formulation.
- Requires no learned value model.
- Useful for validating the theory.

Disadvantages:

- Computationally expensive.
- Hard to deploy in real-time decoding.
- Quality depends heavily on the classifier and rollout budget.

### Variant B: Learned survival value model

Train a small model $V_\phi(s,t)$ or $h_\phi(s,t)$ to predict future safety survival.

The target is:
$$h_t(s)
=
\sum_v
p(v \mid s)
\mathbf{1}\{sv \notin H\}
h_{t+1}(sv).$$


In practice, generate rollouts from the base LM, label them with a streaming safety classifier, and train the survival model to predict whether the rollout will eventually hit the harmful set.

A log-space model may be preferable:
$$V_\phi(s,t) \approx \log h_t(s).$$


Then decoding uses:
$$\log q_t(v \mid s)
\propto
\log p_t(v \mid s)
+
V_\phi(sv,t+1).$$

Advantages:

- Efficient at inference time.
- Directly estimates the Doob \(h\)-function.
- Can generalize beyond sampled rollouts.

Disadvantages:

- Approximation error may break the exact conditioning guarantee.
- Requires generating and labeling training data.
- The learned model may inherit classifier blind spots.

### Variant C: Hazard-based soft survival model

Instead of treating harmfulness as a hard absorbing set, define a hazard function:
$$\lambda(s) = \Pr(s \in H \mid s).$$


Then approximate survival using accumulated future hazard:
$$h(s)
\approx
\mathbb{E}_{p_{\mathrm{LM}}}
\left[
\exp
\left(
-\sum_{u=t}^{T}
\lambda(y_{\le u})
\right)
\mid s
\right].$$

This gives a soft version of the killed-process formulation.

Advantages:

- More robust to noisy classifiers.
- Allows graded risk rather than binary safety.
- May work better for ambiguous or context-dependent safety boundaries.

Disadvantages:

- Less pure than the hard Doob killed-process construction.
- The interpretation becomes risk-sensitive control rather than strict conditioning on survival.

## Main theoretical claim

If $h$ is exact and $H$ is the true harmful prefix set, then the Doob-transformed process samples exactly from the base LM conditioned on avoiding $H$ until horizon $T$:
$$q(y_{1:T})
=
p_{\mathrm{LM}}(y_{1:T} \mid \tau_H > T).$$

Therefore:

1. The transformed process assigns zero probability to trajectories that enter $H$.
2. The transformation minimally distorts the base LM in the sense that it samples from the conditional distribution induced by the original LM.
3. The method avoids harmful regions before entering them, rather than waiting for harmful text to appear and then filtering or refusing.

The guarantee is only as strong as the harmful set $H$. If the classifier misdefines $H$, the safety guarantee applies only to the classifier-defined harmful region.

## Relation to prior work

### FUDGE

FUDGE uses a discriminator over partial sequences to guide generation toward a desired final attribute. It is structurally close because it estimates future attribute satisfaction from prefixes.

However, FUDGE is framed as conditional attribute generation, not as a killed stochastic process conditioned on never entering an unsafe prefix set.

The proposed method differs by making survival avoidance the central object.

### Twisted Sequential Monte Carlo for language models

Twisted SMC methods learn twist functions that estimate future potentials and use them to guide sequence sampling.

This is probably the closest probabilistic ancestor.

However, the proposed method is specialized to inference-time safety and uses the killed-process / survival-harmonic interpretation explicitly.

### DExperts

DExperts uses expert and anti-expert models to modify decoding probabilities, including for detoxification.

It is an important inference-time safety baseline, but it does not estimate a survival harmonic function and is not framed as conditioning the base process on avoiding a harmful set.

### ShieldHead and token-level safety classifiers

Recent safety-decoding work uses lightweight classifiers or heads to detect harmful candidate tokens or unsafe model states during generation.

These approaches are directly relevant but appear more local or myopic.

The proposed method differs by asking whether a candidate token leads to future survival, not merely whether it is immediately unsafe.

### Grammar-constrained decoding and Doob transforms

Recent grammar-constrained decoding work explicitly connects constrained autoregressive generation with Doob $h$-transforms.

This is conceptually useful because it shows that exact constrained decoding requires a future reachability or survival term.

The proposed method transfers that logic from formal grammar constraints to semantic safety constraints.

### Diffusion language models

Some recent diffusion-LM work explicitly uses Doob $h$-transform language for guided denoising or token ordering.

This supports the broader relevance of the Doob framing, but the proposed method targets autoregressive LLM safety at inference time.

## Plausible novelty

The following combination appears plausibly novel:
$$\text{autoregressive LM}
+
\text{classifier-defined harmful prefix set}
+
\text{killed process}
+
\text{survival harmonic function}
+
\text{inference-time Doob transform}.$$

Several components exist separately:

- classifier-guided decoding,
- future discriminators,
- twisted SMC,
- toxicity-aware decoding,
- grammar-conditioned Doob decoding,
- reward-guided diffusion or LM sampling.

But the specific framing of safety as a killed-process Doob transform conditioned on never entering a harmful prefix set appears less explored.

A careful novelty claim should be:

> We formulate inference-time LLM safety as a Doob \(h\)-transform of an autoregressive process killed on entry to a classifier-defined harmful prefix set. Unlike local safety filters, our method estimates future survival probabilities and samples from the base LM conditioned on remaining in the safe region.

Avoid claiming:

> We invented classifier-guided decoding.

That is too broad and already covered by prior work.

## Experimental plan

### Models

Possible base models:

- Llama-family instruction models
- Mistral-family instruction models
- Qwen-family instruction models
- A smaller open model for cheap ablations

### Safety classifiers

Possible classifiers:

- A standalone moderation model
- A fine-tuned harmfulness classifier
- A lightweight hidden-state safety head
- An LLM-as-judge classifier for offline labeling
- A combination of binary harmfulness and graded hazard scores

### Baselines

Compare against:

1. Base LM without safety decoding.
2. Hard output filtering.
3. Local classifier logit penalty.
4. Rejection sampling.
5. FUDGE-style discriminator guidance.
6. DExperts-style detoxification.
7. Token-level safety head or ShieldHead-style method.
8. Refusal-prompt baseline.
9. Rollout-based oracle survival decoding.

### Metrics

Safety:

- Attack success rate.
- Harmful completion rate.
- Toxicity score.
- Jailbreak success rate.
- Classifier-defined hitting probability.

Helpfulness:

- Task success.
- Answer relevance.
- Win rate judged by an LLM or humans.
- Length-normalized helpfulness.

Over-refusal:

- False refusal rate.
- Performance on safe-but-sensitive prompts.
- XSTest-style exaggerated safety evaluation.

Distributional distortion:

- KL divergence from base LM over non-harmful completions.
- Perplexity under base LM.
- Diversity metrics.
- Semantic similarity to unconstrained safe completions.

Latency and compute:

- Decoding overhead.
- Number of classifier calls.
- Number of survival-model calls.
- Rollout budget if applicable.

### Benchmarks

Potential benchmarks:

- HarmBench for harmful behavior and robust refusal.
- ToxicChat for real-world user-AI toxicity moderation.
- XSTest for exaggerated safety / over-refusal.
- AdvBench or jailbreak-style prompt sets.
- RealToxicityPrompts for toxicity continuation.
- Custom prefix-hazard benchmark for measuring early avoidance.

## Key ablations

1. Local classifier only vs survival classifier.
2. Hard mask only vs hard mask plus $h$-reweighting.
3. Learned \(h\) vs rollout-estimated $h$.
4. Binary survival vs soft hazard survival.
5. Different horizons $T$.
6. Different top-$K$ candidate sets for reweighting.
7. Different safety classifiers.
8. Calibration of $\hat h$.
9. Effect on over-refusal.
10. Effect on benign but safety-sensitive prompts.

## Possible failure modes

### Classifier misspecification

If the classifier fails to identify harmful prefixes, the Doob transform cannot avoid them.

### Over-conservatism

If the classifier marks too many prefixes as dangerous, the model may become evasive or over-refuse.

### Survival-model miscalibration

If $\hat h$ is poorly calibrated, the decoder may avoid harmless regions or enter unsafe ones.

### Adversarial prefixes

Jailbreak prompts may exploit blind spots in the classifier or survival estimator.

### Horizon mismatch

A finite horizon \(T\) may miss harmful continuations beyond the chosen decoding window.

### Degenerate safe attractors

The model may learn to steer toward generic disclaimers, refusals, or low-information completions because they have high estimated survival probability.

## Practical implementation notes

At each step, it is not necessary to evaluate all vocabulary tokens.

A feasible implementation can:

1. Take the top-$K$ tokens under the base LM.
2. Remove tokens whose candidate prefix is immediately harmful.
3. Score the remaining candidates using $\log p(v \mid s) + \beta \log \hat h(sv)$.
4. Renormalize and sample.

The coefficient $\beta$ can control safety strength:
$$\log q(v \mid s)
\propto
\log p(v \mid s)
+
\beta \log \hat h(sv).$$

For the exact Doob transform, $\beta = 1$. In practice, tuning $\beta$ may improve safety-helpfulness tradeoffs.

A temperature parameter can also be applied after the transformed logits.

## Possible paper structure

### Abstract sketch

We propose an inference-time decoding method for language model safety based on the Doob $h$-transform. We model unsafe generation as a killed autoregressive process, where trajectories terminate upon entering a classifier-defined harmful prefix set. The exact safe sampler is obtained by conditioning the base language model on survival, yielding a next-token distribution reweighted by a harmonic survival function. We approximate this function using rollout estimates or a learned survival model. Unlike local safety filters, our method anticipates future unsafe continuations and steers generation away from harmful basins before harmful text is emitted. Experiments evaluate safety, helpfulness, over-refusal, and distributional distortion against classifier-guided decoding, rejection sampling, DExperts, and future-discriminator baselines.

## Bottom-line claim

The idea is credible and worth developing.

The clean contribution is not "classifier-guided safe decoding" in general, but rather:

> A principled Doob \(h\)-transform formulation of inference-time LLM safety, where harmful generation is modeled as a killed process and decoding is conditioned on survival.

This gives a strong theoretical foundation, a clear algorithmic path, and a plausible novelty gap relative to existing decoding-time safety methods.

## Citations

[1]: Yang, K., & Klein, D. (2021). FUDGE: Controlled Text Generation With Future Discriminators. NAACL 2021. https://aclanthology.org/2021.naacl-main.276/

[2]: Zhao et al. (2024). Probabilistic Inference in Language Models via Twisted Sequential Monte Carlo. arXiv. https://arxiv.org/abs/2404.17546

[3]: Liu et al. (2021). DExperts: Decoding-Time Controlled Text Generation with Experts and Anti-Experts. ACL 2021. https://aclanthology.org/2021.acl-long.522/

[4]: ShieldHead: Early Risk Detection in Language Model Decoding via Internal Classifier Heads. ACL Findings 2025. https://aclanthology.org/2025.findings-acl.932/

[5]: Alpay & Senturk. Grammar-Aligned Decoding and Doob \(h\)-Transform Characterizations for Constrained Autoregressive Generation. arXiv. https://arxiv.org/html/2603.05540v1

[6]: DPRM: Doob \(h\)-Transform / Process-Reward Methods for Diffusion Language Models. arXiv. https://arxiv.org/abs/2604.24357

[7]: HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal. https://www.harmbench.org/

[8]: ToxicChat: Unveiling Hidden Challenges of Toxicity Detection in Real-World User-AI Conversation. https://arxiv.org/abs/2310.17389

[9]: XSTest: A Test Suite for Identifying Exaggerated Safety Behaviors in Large Language Models. https://arxiv.org/abs/2308.01263