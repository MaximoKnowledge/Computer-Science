**Abstract**: We study flow-map models trained with Lagrangian, Eulerian, and semigroup consistency losses. Although these losses target the same ideal flow map, they differ in training stability and downstream performance. We propose that these differences arise because the objectives induce different internal representations of the same object. To test this, we perform a mechanistic analysis using residual tomography, representation similarity metrics, linear probes, local geometry diagnostics, activation patching, and cross-model vector swaps. The goal is to explain how each objective represents transport, time, velocity, and composition, and to determine whether performance differences can be attributed to objective-induced representational structure rather than only optimizer behavior.
## Core Idea

We study whether different self-distillation losses for learning flow maps induce different internal representations, even when the architecture, parameterization, data distribution, and diagonal flow-matching objective are held fixed.

The starting point is the flow-map parameterization

$$  
X_{s,t}(x) = x + (t-s)v_{s,t}(x),  
$$

where ($X_{s,t}$) maps a point at time (s) to its corresponding point at time (t) along the probability-flow trajectory. Existing work proposes multiple mathematically equivalent characterizations of the same ideal flow map, including Lagrangian, Eulerian, and semigroup/progressive consistency conditions. In the infinite-data, infinite-capacity, globally optimized setting, these objectives should recover the same object. In practice, however, they lead to different training dynamics and different downstream performance.

The central hypothesis is that these differences are not merely optimizer artifacts. Instead, each loss induces a different representational bias over how the network encodes time, transport, local velocity, and compositional consistency.

We propose a mechanistic interpretability study of flow-map models trained with different consistency losses. The goal is to identify how Lagrangian, Eulerian, and semigroup-based training objectives organize the internal computation of the same mathematical object.

## Motivation

Flow-map models are appealing because they amortize ODE integration: rather than repeatedly evaluating a velocity field during sampling, they learn a map ($X_{s,t}$) that can jump directly between times. This enables fast one-step or few-step generation.

Recent work shows that the same flow-map parameterization can be trained using different consistency objectives:
- a Lagrangian condition, enforcing that the transported point follows the learned diagonal velocity;
- an Eulerian condition, enforcing a PDE constraint involving spatial derivatives of the map;
- a semigroup or progressive consistency condition, enforcing that one large jump equals a composition of smaller jumps.

Empirically, these objectives behave differently. Lagrangian training is often more stable and performs better; Eulerian training can be unstable because of spatial derivative terms; progressive or semigroup training can suffer from bootstrapping and compounding errors.

The usual explanation is optimization: some losses are easier to train than others. That explanation is plausible but incomplete. These losses do not merely change gradient statistics; they expose the model to different constraints, different credit-assignment paths, and different notions of consistency. Therefore, they may cause the model to represent the flow map in qualitatively different ways.

The proposed project asks:

> When two models learn the same flow map through different mathematical characterizations, what internal representations do they actually learn?

## Main Hypothesis

Although Lagrangian, Eulerian, and semigroup objectives share the same ideal solution, they induce different internal computational strategies.

A possible working hypothesis is:
1. **Lagrangian-trained models** learn more trajectory-aligned representations. Their internal states should better encode transported points, endpoint displacement, and diagonal velocity consistency.
2. **Eulerian-trained models** learn more local differential representations. Their hidden states may encode spatial sensitivity, Jacobian structure, and local PDE residual information, but may also show higher gradient instability.
3. **Semigroup/progressive-trained models** learn more compositional representations. Their hidden states may better encode step composition and time-interval decomposition, but may accumulate errors when composing out-of-distribution intermediate jumps.

The project is not merely to compare sample quality. The goal is to identify which internal features are responsible for satisfying, or failing to satisfy, each mathematical condition.

## Problem Setup

We train multiple flow-map models with identical architecture, data, interpolant, optimizer budget, and parameterization:

$$  
X_{s,t}^{\theta}(x) = x + (t-s)v_{s,t}^{\theta}(x).  
$$

All models share the same diagonal flow-matching objective, which trains

$$  
v_{t,t}^{\theta}(x) \approx b_t(x).  
$$

They differ only in the off-diagonal consistency objective:
$$  
\mathcal{L}_{\text{total}}

\mathcal{L}_{\text{diag}}  
+  
\mathcal{L}_{\text{consistency}}.  
$$

The consistency term is chosen from:

### Lagrangian Consistency

The Lagrangian objective enforces

$$  
\partial_t X_{s,t}(x)  
\approx  
v_{t,t}(X_{s,t}(x)).  
$$

This says that if the model jumps from ($s$) to ($t$), the endpoint should move according to the diagonal velocity at time ($t$).

### Eulerian Consistency

The Eulerian objective enforces

$$  
\partial_s X_{s,t}(x)  
+  
\nabla X_{s,t}(x) v_{s,s}(x)  
\approx  
0.  
$$

This is a PDE-style constraint over the flow map. It explicitly involves spatial derivatives of ($X_{s,t}$), which may encourage representations sensitive to local geometry and Jacobian structure.

### Semigroup / Progressive Consistency

The semigroup objective enforces

$$  
X_{s,t}(x)  
\approx  
X_{u,t}(X_{s,u}(x)).  
$$

This says that a direct jump from ($s$) to ($t$) should agree with a composed jump through an intermediate time ($u$).

## Core Mechanistic Question

The core question is:

> Do different consistency losses cause the model to represent the same flow map through different internal mechanisms?

More concretely:
- Does the Lagrangian model encode trajectory-local velocity more directly?
- Does the Eulerian model encode spatial derivative information more strongly?
- Does the semigroup model encode time-compositional structure more strongly?
- Are downstream performance differences explained by measurable differences in representation geometry, causal feature usage, or consistency-error localization?
- Can internal representations trained under one objective be transferred into another model without breaking the corresponding mathematical constraint?

## Experimental Plan

### 1. Consistency Residual Tomography

For each trained model, evaluate all three consistency residuals, not only the one it was trained on.

For a grid of ($(s,t,u)$), compute: 
$$  
R_{\text{Lag}}(s,t,x)
=
 \left|  
\partial_t X_{s,t}(x)
-
v_{t,t}(X_{s,t}(x))  
\right|,  
$$

$$  
R_{\text{Eul}}(s,t,x)
=
\left|  
\partial_s X_{s,t}(x)  
+  
\nabla X_{s,t}(x)-v_{s,s}(x)  
\right|,  
$$

$$  
R_{\text{Semi}}(s,u,t,x) = \left|  
X_{s,t}(x)
-
X_{u,t}(X_{s,u}(x))  
\right|.  
$$

This gives a residual profile for each model over time intervals, spatial regions, and trajectory positions.

The first diagnostic is whether each model only satisfies the constraint it was trained on, or whether some objectives induce broader consistency across all characterizations.

### 2. Representation Similarity Across Time

For each layer ($l$), time pair ($(s,t)$), and input batch ($x_s$), collect hidden states

$$  
h_l^{\text{method}}(s,t,x_s).  
$$

Then compute CKA, SVCCA, PWCCA, or Procrustes-aligned similarity across:
- times within the same model;
- layers within the same model;
- models trained with different losses;
- checkpoints during training.

Important comparisons include:

$$  
h_l^{\text{LSD}}(s,t,x)  
\quad \text{vs.} \quad  
h_l^{\text{PSD}}(s,t,x),  
$$

and

$$  
h_l(s,t,x)  
\quad \text{vs.} \quad  
h_l(s,u,x), \quad h_l(u,t,X_{s,u}(x)).  
$$

This tests whether semigroup-trained models organize representations compositionally across time, while Lagrangian models organize them more continuously along trajectories.

### 3. Causal Activation Patching Within a Model

Within a single trained model, patch activations across time pairs.

For example, run the model on ($(s,t,x)$), but replace an intermediate activation with the activation from ($(s,u,x)$) or ($(u,t,X_{s,u}(x))$). Then measure changes in:
- output flow-map error;
- Lagrangian residual;
- Eulerian residual;
- semigroup residual;
- sample quality;
- endpoint displacement direction.

This gives a causal measure of whether particular layers encode interval-specific or compositional information.

A key experiment is:

> Does patching semigroup-consistent intermediate activations improve or preserve semigroup consistency more in PSD-trained models than in LSD-trained models?

### 4. Cross-Model Activation Patching

Train models with identical initialization and data order, changing only the consistency loss. Then patch activations from one model into another.

Examples:
- patch LSD activations into PSD model;
- patch PSD activations into LSD model;
- patch ESD activations into LSD model;
- patch diagonal-time activations from one model into off-diagonal computation of another.

The outcome is measured not only by output error, but by which mathematical residual is preserved or broken.

This tests whether representations are functionally compatible across objectives.

Because independently trained networks may use different bases, this experiment should be run in two regimes:

1. shared initialization and synchronized training setup;
2. post-hoc aligned representations using Procrustes or learned linear maps.

### 5. Vector-Swap Experiments at the Flow-Map Level

In addition to hidden-state patching, directly swap predicted displacement vectors
$$  
v_{s,t}(x)  
$$

or intermediate transported points
$$  
X_{s,u}(x)  
$$
across models.
For example: 
$$  
\tilde{X}_{s,t}(x) =
x + (t-s)v_{s,t}^{\text{LSD}}(x),  
$$

inside a PSD composition test:
$$  
X_{u,t}^{\text{PSD}}(\tilde{X}_{s,u}(x)).  
$$
This asks whether the explicit learned vector field from one objective can serve as a valid component inside another objective’s computation.

Possible outcomes:
- LSD vectors transfer well into Lagrangian residuals but poorly into semigroup compositions.
- PSD vectors transfer well for interval composition but poorly for diagonal velocity matching.
- ESD vectors show stronger local sensitivity but worse long-range compositional stability.

### 6. Linear Probes for Mathematical Quantities

Train simple probes on hidden states to predict quantities such as:
- diagonal velocity ($b_t(x)$);
- endpoint ($X_{s,t}(x)$);
- displacement ($X_{s,t}(x)-x$);
- time interval length ($t-s$);
- Lagrangian residual;
- Eulerian residual;
- semigroup residual;
- local Jacobian norm;
- trajectory curvature.

The point is not to improve the model, but to identify what information is linearly accessible in each objective’s representation.

A useful hypothesis is:
- Lagrangian models make velocity and endpoint information more linearly accessible.
- Eulerian models make Jacobian or spatial sensitivity information more accessible.
- Semigroup models make interval length and compositional structure more accessible.

### 7. Local Geometry and Sensitivity Analysis

For each model, measure local linearization properties of the learned map:
$$  
J_{s,t}(x) = \nabla_x X_{s,t}(x).  
$$
Track:
- singular value spectrum of ($J_{s,t}$);
- Jacobian norm;
- divergence;
- local Lipschitz estimates;
- sensitivity to perturbations along and orthogonal to the trajectory;
- alignment between perturbation response and learned velocity.

This is especially important for comparing Eulerian and Lagrangian training, because the Eulerian objective explicitly involves spatial derivatives, while the Lagrangian objective avoids them.

### 8. Training-Dynamics Analysis
Store checkpoints throughout training and repeat the representation and residual analyses over time.

This allows us to ask:
- Which consistency condition emerges first?
- Does diagonal velocity learning precede off-diagonal consistency?
- Do PSD models first learn short intervals and later extend to long intervals?
- Do LSD models propagate diagonal information more smoothly into the off-diagonal region?
- Does ESD instability appear first in Jacobian spectra, residual spikes, or hidden-state drift?

This converts the project from a static comparison into a mechanistic study of how the objectives shape learning dynamics.

## Evaluation Metrics
The evaluation should separate generative performance from mechanistic structure.
### Generative Metrics
- FID or dataset-specific generation metric;
- one-step and few-step sample quality;
- quality as a function of number of sampling steps;
- diversity and mode coverage;
- endpoint distribution error on low-dimensional data.

### Consistency Metrics
- Lagrangian residual;
- Eulerian residual;
- semigroup residual;
- diagonal flow-matching error;
- off-diagonal extrapolation error;
- error as a function of interval length (t-s);
- error as a function of intermediate composition point (u).

### Representation Metrics
- CKA / SVCCA / PWCCA across models;
- Procrustes-aligned similarity;
- layerwise time similarity;
- hidden-state smoothness over ((s,t));
- representation rank and effective dimension;
- linear probe accuracy for velocity, endpoint, residuals, and time interval.

### Causal Metrics
- change in output after activation patching;
- change in PDE residual after activation patching;
- cross-model patching compatibility;
- vector-swap degradation;
- layerwise causal importance for each consistency condition.

## Expected Outcomes

The best possible outcome is not merely that one method has better FID. The stronger result would be a mechanistic explanation of why the methods differ.

Possible findings include:

1. **Lagrangian training produces trajectory-aligned representations.**  
    Hidden states may vary smoothly with (t), encode diagonal velocity strongly, and support stable long-range jumps.
2. **Eulerian training produces derivative-sensitive representations.**  
    These may better encode local spatial geometry, but exhibit high Jacobian norms, unstable gradients, or fragile hidden-state geometry.
3. **Semigroup training produces compositional but bootstrapped representations.**  
    These may support short-step composition but show error accumulation or distribution shift for long jumps.
4. **Cross-loss models learn partially incompatible internal bases.**  
    Even when the output parameterization is identical, internal features may not be directly swappable without alignment.
5. **Performance differences can be localized to specific time regions.**  
    For example, PSD may fail primarily for large (t-s), while ESD may fail in high-curvature regions of the data manifold.

## Feasibility
The project is feasible, especially if staged carefully.
The first stage should use a low-dimensional dataset such as checkerboard or Gaussian mixtures. This allows exact visualization of trajectories, residuals, vector fields, Jacobians, and semigroup errors.
The second stage can move to small image models, where activation analysis becomes more meaningful but also more expensive.
The third stage can test whether the same mechanistic signatures persist in larger image models.

The main engineering requirements are:
- access to trained LSD, ESD, and PSD models under matched conditions;
- hooks for hidden-state extraction;
- automatic differentiation for time and spatial derivatives;
- consistent sampling over ($(s,t,u)$);
- checkpoint storage;
- activation patching infrastructure.

The main interpretability risk is that cross-model activation patching may be confounded by representation misalignment. This can be mitigated through shared initialization, synchronized training, and post-hoc linear alignment.

## Main Claim

The main claim should be stated carefully:
> Different self-distillation losses for flow-map learning induce measurably different internal representations of time, transport, and consistency, even when they target the same ideal flow map.

A stronger claim, conditional on positive evidence, would be:
> The empirical performance gap between Lagrangian, Eulerian, and semigroup-trained flow maps can be partially explained by objective-specific representational mechanisms, not only by generic optimizer instability.

The claim should not be:
> One loss is universally better because its samples have better FID.

The mechanistic claim is about how the learned computation differs.

## Key Contributions

1. A mechanistic interpretability framework for flow-map models trained under different mathematical consistency objectives.
2. A residual-tomography analysis that evaluates Lagrangian, Eulerian, and semigroup consistency across all models, not only the residual used during training.
3. A representation-similarity study of flow-map hidden states across time, layer, and objective.
4. A causal intervention suite, including activation patching, vector swaps, and cross-model representation exchange.
5. A set of diagnostics connecting internal representations to mathematical properties of learned flow maps, including velocity alignment, local Jacobian geometry, and compositional consistency.