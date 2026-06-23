## Abstract

Continuous-time diffusion and flow-based generative models are usually justified through population-level stochastic or deterministic dynamics: a forward noising process defines a family of marginals, and generation is obtained by integrating a learned reverse-time score or velocity field over a continuous interval. In theory, if the learned score or vector field is correct for all times, the resulting reverse SDE/ODE recovers the target distribution. In practice, however, training only observes finite data, finite Monte Carlo samples of time, finite optimization, and a finite-capacity time-conditioned neural network. This raises a representation-level question: does the trained model actually learn a coherent continuous-time field, or does it learn a high-quality finite-time surrogate that performs well on the sampled training-time distribution but does not faithfully represent the intended SDE/ODE?

We propose to study this failure mode as a temporal amortization gap: the discrepancy between treating a time-conditioned neural network as a continuous-time score/vector field and what is actually certified by finite-budget training. The central hypothesis is that standard denoising score matching, flow matching, or ELBO-style objectives validate marginal regression at sampled times, but do not necessarily certify continuous-time generalization, off-grid consistency, solver-transfer robustness, or realizability with respect to the intended noising process. This distinction matters because the convergence of continuous-time generative models to their target dynamics is typically stated under a perfect-score or small-score-error assumption, while practical training only supplies an approximate learned representation.

The goal is not to claim that trained models fail to define any SDE or ODE. Once a learned field is inserted into a reverse-time equation, it defines a neural SDE/ODE under suitable regularity conditions. The question is sharper: does this learned equation correspond to the intended reverse dynamics induced by the forward noising kernels, or is it merely a discretization- and schedule-adapted surrogate? We aim to expose, measure, and reduce this gap using held-out-time evaluation, grid-refinement tests, solver-transfer diagnostics, time-embedding ablations, and residual-based consistency checks.

## Core Research Question

Do finite-budget time-conditioned diffusion and flow models learn a genuinely continuous-time score/vector-field representation, or do they primarily amortize performance over the finite distribution of timesteps encountered during training?

Equivalently:

- Does low training or validation loss over sampled timesteps imply meaningful off-grid temporal generalization?
- Does a model trained with one time-sampling distribution remain robust under different solver grids, timestep schedules, and quadrature rules?
- Can two models with similar denoising/flow-matching loss differ substantially in whether they behave like coherent continuous-time dynamical systems?
- How do architecture, time embeddings, optimization budget, data budget, and timestep sampling affect this gap?

## Proposed Terminology

### Temporal Amortization Gap

The gap between the continuous-time object assumed by theory and the finite-time representation learned by a time-conditioned neural network under finite training.

Informally:

$TAG = performance\ on\ the\ intended\ continuous\ time\ field - performance\ on\ the\ observed\ training-time\ distribution$

A practical version compares error or behavior on held-out timesteps, alternative time distributions, refined grids, and solver schedules.

### Continuous-Time Realization Gap

The gap between the fact that a learned field can be inserted into an SDE/ODE and the stronger claim that it realizes the intended reverse dynamics associated with the forward noising process.

A learned score $s_\theta(x,t)$ may define a reverse-time neural SDE, but it need not approximate the true score $\nabla_x \log p_t(x)$ uniformly or coherently across $t$.

### Time-Field Generalization

The ability of a time-conditioned model to generalize as a function of time, not merely as a function of data/noise pairs.

This asks whether the learned representation behaves like a field over $[0,1]$, rather than like a flexible lookup or interpolation mechanism over training-exposed noise levels.

## Related Work

### Score-Based Generative Modeling through SDEs

Song et al. formulate diffusion models as continuous-time SDEs. A forward SDE gradually transforms data into noise, and a reverse-time SDE can generate data if the time-dependent score $\nabla_x \log p_t(x)$ is known or accurately estimated. This provides the theoretical basis for treating diffusion models as continuous-time dynamical systems.

Relevance: This is the foundational framework whose practical assumption we want to interrogate. The theory requires a score field over time, but finite training only supervises sampled times.

### Denoising Score Matching and Continuous-Time Objectives

Continuous-time diffusion training usually optimizes an expectation over time, data, and perturbation noise. In implementation, this expectation is approximated by finite Monte Carlo samples. Thus, the learned model is only directly constrained at the sampled training distribution over time.

Relevance: The proposed project focuses on the difference between the population objective and what finite neural training actually certifies.

### FP-Diffusion and Score Fokker-Planck Consistency

FP-Diffusion shows that scores learned by denoising score matching can fail to satisfy the underlying score Fokker-Planck equation, even when empirical generation quality is strong. The method adds a regularizer that encourages the learned score to obey the PDE structure satisfied by the true score field.

Relevance: This is one of the closest prior works. It shows that good DSM performance does not necessarily imply dynamical self-consistency. However, the proposed temporal amortization view is more representation-level: it asks whether the time-conditioned network has learned a coherent continuous-time object in the first place.

### Closing the ODE-SDE Gap

Work on the ODE-SDE gap studies the discrepancy between probability-flow ODE sampling and reverse-SDE sampling under learned neural scores. It connects differences between the induced ODE and SDE distributions to Fokker-Planck residuals.

Relevance: This supports the view that the neural approximation induces its own dynamics, which may differ from the ideal continuous-time theory. The proposed project can use solver-transfer and ODE/SDE discrepancy as symptoms of temporal realization failure.

### Consistent Diffusion Models and Sampling Drift

Consistent Diffusion Models address the fact that standard DSM trains on non-drifted noisy data, while sampling recursively feeds the model its own generated states. They enforce consistency properties across time and generated states to reduce sampling drift.

Relevance: This is adjacent but not identical. Sampling drift concerns distribution shift along the generated trajectory. Temporal amortization concerns whether the learned field generalizes over time and solver queries as a continuous-time representation.

### Timestep Embedding and Time Awareness

Recent work on timestep embeddings shows that architectural choices can weaken or even eliminate effective time awareness in time-dependent neural networks. This suggests that merely providing $t$ as an input does not guarantee that the network uses it correctly or robustly.

Relevance: This is highly relevant to the representation-level question. The proposed project can study whether different time embeddings encourage true continuous-time generalization or allow schedule-specific memorization/interpolation.

### Flow Matching and Independent Timestep Training

Flow matching and rectified-flow methods learn time-dependent velocity fields, often using objectives that treat timesteps independently. Temporal Pair Consistency and related work attempt to couple predictions across times along the same probability path.

Relevance: This is close in spirit for flow models. The proposed project can extend the question beyond diffusion scores to velocity fields: does the learned vector field behave coherently over continuous time, or only over the training-time distribution?

### Score Estimation and Generalization Theory

Several theoretical works bound sampling error under assumptions on learned-score accuracy, often requiring small integrated score error over time. More recent work studies finite-sample, neural-network-based, or optimization-aware score estimation.

Relevance: These works clarify what assumptions are needed for convergence, but they typically do not isolate the practical representation question: whether a finite trained network has learned the correct continuous-time field rather than a finite-time surrogate.

## Novelty Claim

The proposed contribution should not be framed as:

- "Diffusion models are trained with finite timesteps."
- "The learned score may violate the Fokker-Planck equation."
- "Imperfect score matching causes sampling errors."
- "Timestep embeddings matter."

Those are already known or closely studied.

The stronger and more novel framing is:

Continuous-time diffusion and flow models rely on a hidden temporal representation generalization assumption. Although theory treats the learned score or velocity as a continuous-time field, practical training only constrains a finite-budget, time-conditioned regression problem. We study whether the resulting neural representation behaves as a coherent continuous-time dynamical object, or as a high-quality finite-time surrogate adapted to the training schedule.

## Possible Contributions

1. Formalize the temporal amortization gap.

Define metrics that distinguish ordinary validation loss from continuous-time field generalization. These may include held-out-time error, off-grid score/velocity error, PDE residuals, solver-transfer degradation, and sensitivity to timestep distributions.

2. Build controlled benchmarks.

Use synthetic distributions where the true score or velocity is known, allowing exact evaluation of temporal generalization. Examples include Gaussian mixtures, low-dimensional manifolds, analytically tractable SDEs, and controlled probability paths for flow matching.

3. Test off-grid and held-out-time behavior.

Train models using restricted, structured, or biased timestep distributions, then evaluate on held-out intervals, interleaved time grids, or continuous random time queries.

4. Study solver-transfer robustness.

Train a model under one timestep distribution or sampler schedule, then evaluate generation under alternative valid solvers, refined grids, coarser grids, adaptive solvers, and shifted noise schedules.

5. Analyze architecture and time embeddings.

Compare sinusoidal embeddings, learned discrete embeddings, Fourier features, low-frequency embeddings, spline-based time parameterizations, monotone encodings, and time-smooth architectures.

6. Connect representation failure to sample quality.

Show whether temporal amortization diagnostics predict FID, likelihood, precision/recall, ODE/SDE gap, few-step sampling degradation, or distillation instability better than standard validation losses.

7. Propose regularizers or architectural constraints.

Develop methods that encourage continuous-time coherence, such as temporal smoothness penalties, score-FPE/continuity residuals, paired-time consistency, derivative constraints with respect to time, or restricted time embeddings.

## Candidate Experiments

### Experiment 1: Held-Out Timestep Intervals

Train using timesteps sampled from a subset of $[0,1]$, such as alternating intervals or a sparse grid with jitter. Evaluate denoising/score/velocity error on both seen and unseen time regions.

Expected finding:
A model may show low training-time loss but high off-grid or held-out-time error, especially with high-capacity time embeddings or limited training budgets.

### Experiment 2: Grid Refinement Test

Train using a fixed or biased timestep distribution. Sample using increasingly fine solver grids. If the model learned a true continuous-time field, behavior should improve or degrade predictably according to solver error. If it learned a schedule-specific surrogate, finer or shifted grids may not help and may even hurt.

Expected finding:
Generation quality may be non-monotonic under grid refinement when the model lacks continuous-time coherence.

### Experiment 3: Solver Transfer

Train under one sampler-compatible schedule and evaluate under another, such as Euler-Maruyama, probability-flow ODE, predictor-corrector, adaptive ODE, or alternative discretizations.

Expected finding:
Models with similar validation loss may differ significantly in solver-transfer robustness.

### Experiment 4: Time Embedding Capacity

Compare time embeddings of different capacity and inductive bias. High-capacity learned embeddings may support timestep-specific fitting, while smoother or lower-frequency parameterizations may improve temporal generalization.

Expected finding:
Architecture can control the temporal amortization gap independently of ordinary training loss.

### Experiment 5: PDE or Continuity Residual Evaluation

For diffusion models, evaluate score-Fokker-Planck residuals. For flow matching, evaluate continuity-equation consistency or paired-time trajectory consistency.

Expected finding:
Residual-based diagnostics may reveal failures that are invisible under DSM or flow-matching validation loss.

### Experiment 6: Budget Scaling

Vary data size, number of unique timesteps, number of optimization steps, model size, and compute. Measure whether temporal generalization improves smoothly or whether models overfit the training-time distribution under realistic budgets.

Expected finding:
The continuous-time assumption may become more valid with scale, but the rate and conditions under which this happens are empirical questions.

## Do's

- Frame the project as a representation and generalization problem, not merely a discretization problem.
- Distinguish carefully between "the learned field defines some neural SDE/ODE" and "the learned field realizes the intended reverse SDE/ODE."
- Use controlled settings where the true score or velocity is known.
- Evaluate on held-out times, not only held-out data.
- Test solver transfer and grid refinement, because these directly probe continuous-time semantics.
- Compare models with matched validation loss but different temporal behavior.
- Include architecture and time-embedding ablations.
- Connect diagnostics to practical outcomes such as sample quality, likelihood, ODE/SDE gap, or few-step sampling.
- Discuss Fokker-Planck and continuity-equation residuals as related but not identical to the proposed gap.
- Make the claim falsifiable: identify conditions under which standard training does appear to learn a robust continuous-time field.

## Don'ts

- Do not claim that diffusion models do not define an SDE. A learned score inserted into a reverse-time equation can define a neural SDE under suitable regularity assumptions.
- Do not claim that finite discretization alone invalidates continuous-time diffusion theory. The issue is the gap between ideal population assumptions and finite learned representations.
- Do not present the idea as "nobody has studied consistency over time." FP-Diffusion, ODE-SDE gap work, consistency models, and flow-matching consistency methods are all relevant.
- Do not rely only on FID or sample visuals. These may hide temporal representation failures.
- Do not conflate data memorization with timestep amortization. They may interact, but they are distinct.
- Do not evaluate only the training timestep distribution. That would reproduce the blind spot being studied.
- Do not assume sinusoidal or continuous embeddings guarantee continuous-time learning.
- Do not make the novelty hinge on the statement that standard objectives approximate integrals with Monte Carlo samples. That observation is too broad and already known.
- Do not overclaim theoretical impossibility. The useful claim is empirical and diagnostic: under finite budgets, continuous-time realization is an assumption that should be measured.
- Do not ignore flow models. The same issue applies to time-dependent velocity fields in flow matching and rectified flow.

## Suggested Positioning

A good positioning statement:

"Continuous-time diffusion and flow models are theoretically formulated as learned dynamical systems over a time continuum. However, practical training only constrains a finite-budget, time-conditioned regression problem. We study the temporal amortization gap: the extent to which a trained model behaves as a coherent continuous-time score or velocity field rather than a high-performing finite-time surrogate. Through held-out-time evaluation, solver-transfer tests, grid-refinement diagnostics, and architecture ablations, we expose when the continuous-time interpretation is empirically justified and when it breaks down."

## Suggested Paper Title Options

- Temporal Amortization Gaps in Continuous-Time Generative Models
- Do Diffusion Models Learn Continuous-Time Scores or Finite-Time Surrogates?
- Testing the Continuous-Time Assumption in Diffusion and Flow Matching
- When Time Conditioning Is Not Continuous-Time Learning
- On the Representation Gap Between Neural Scores and Reverse SDEs
- Are Learned Diffusion Scores Continuous-Time Objects?

## Main Takeaway

The project is promising if it is framed as a diagnostic and representation-level study of continuous-time generalization under finite training. The novelty is not that diffusion and flow models are discretized in practice, nor that learned scores can be imperfect. The novelty is to isolate and measure whether the learned time-conditioned representation deserves to be interpreted as the continuous-time field assumed by the SDE/ODE theory.