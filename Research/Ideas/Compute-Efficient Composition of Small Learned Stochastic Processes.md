## Core Idea

We want to study whether a complex generative stochastic process can be represented more efficiently as a composition of several genuinely small learned stochastic processes.

The proposal is not to divide the time interval of a single SDE into smaller segments while reusing the same large score or drift network. Instead, each stage is implemented by a separate low-capacity model and is responsible for learning a simpler distributional transition.

The complete generative process has the form

$$
q_0
\xrightarrow{K_{\theta_1}}
q_1
\xrightarrow{K_{\theta_2}}
\cdots
\xrightarrow{K_{\theta_M}}
q_M \approx p_{\mathrm{data}},
$$

where each $K_{\theta_i}$ is the transition kernel induced by a small learned SDE, such as

$$
dX_t
=
b_{\theta_i}(X_t,t)\,dt
+
\sigma_{\theta_i}(X_t,t)\,dW_t.
$$

Each drift, score, or diffusion model is substantially smaller than the network used by a conventional monolithic diffusion model.

## Research Question

The central question is

> Can a composition of low-capacity learned stochastic processes model a complex target distribution more efficiently than a single high-capacity stochastic process?

Equivalently, we want to compare

$$
q_M
=
K_{\theta_M}
\circ \cdots \circ
K_{\theta_1}
\# q_0
$$

against

$$
\widetilde q
=
K_{\Theta}\#q_0,
$$

where $K_{\Theta}$ is implemented by one large score or drift network.

The comparison must be performed under controlled computational budgets.

## Main Hypothesis

A sequence of small stochastic models may benefit from specialization. Each model only needs to solve a relatively simple local transport problem, while the composition of these simple transports may produce a highly expressive global process.

This would be analogous to depth efficiency in deterministic neural networks:

$$
\text{composition of simple functions}
\quad\text{versus}\quad
\text{one large function}.
$$

Here, however, the objects being composed are stochastic kernels rather than deterministic maps.

The possible advantage must be balanced against the accumulation of approximation and numerical errors across stages.

## What the Proposal Is Not

The proposal is not merely a temporal discretization of one Markov process.

A conventional model may use one large time-dependent network

$$
s_{\Theta}(x,t)
$$

throughout the complete reverse process.

The proposed model instead uses distinct small networks

$$
s_{\theta_1}(x,t),
s_{\theta_2}(x,t),
\ldots,
s_{\theta_M}(x,t),
$$

with each network belonging to a restricted function class and being trained for a different stochastic transport problem.

Although the resulting composition can mathematically be represented as one time-inhomogeneous Markov process, its parameterization, computational structure, and representational properties are different.

## Compute-Aware Comparison

The principal inference-cost comparison is

$$
\sum_{i=1}^{M}
N_i\,C(s_{\theta_i})
\quad\text{versus}\quad
N\,C(s_{\Theta}),
$$

where

- $N_i$ is the number of evaluations used by stage $i$;
- $C(s_{\theta_i})$ is the cost of evaluating the small model at stage $i$;
- $N$ is the number of evaluations used by the monolithic model;
- $C(s_{\Theta})$ is the cost of evaluating the large model.

The modular construction is useful only if specialization compensates for the additional stages.

A positive result should therefore demonstrate

$$
\sum_{i=1}^{M}
N_i\,C(s_{\theta_i})
<
N\,C(s_{\Theta})
$$

at matched generative quality, or better generative quality at matched total compute.

Relevant budgets include

- inference FLOPs;
- wall-clock latency;
- total parameter count;
- peak memory;
- training compute;
- number of score or drift evaluations.

## General Optimization Problem

A compute-regularized formulation is

$$
\min_{M,\{\theta_i\}}
D\left(
K_{\theta_M}
\circ\cdots\circ
K_{\theta_1}
\#q_0,
p_{\mathrm{data}}
\right)
+
\lambda
\sum_{i=1}^{M}
N_i\,C(\theta_i),
$$

where $D$ is a discrepancy between the generated and target distributions.

An equivalent constrained formulation is

$$
\min_{M,\{\theta_i\}}
D\left(q_M,p_{\mathrm{data}}\right)
$$

subject to

$$
\sum_{i=1}^{M}
N_i\,C(\theta_i)
\leq B,
$$

where $B$ is the total inference budget.

The meta-level problem is to determine

- how many stochastic modules should be used;
- how much capacity each module should receive;
- how many solver evaluations each module should receive;
- which intermediate distributions should separate the modules;
- whether the optimal decomposition depends on the data distribution or compute budget.

## Intermediate Distributions

The intermediate distributions are

$$
q_i
=
K_{\theta_i}\#q_{i-1}.
$$

They may initially be prescribed to isolate the effect of composition. In a more ambitious version, they may emerge from joint end-to-end optimization.

The learned intermediate distributions can be interpreted as latent representations in probability space. A useful decomposition might assign different stages to different aspects of the target distribution, such as

$$
\text{mode allocation}
\rightarrow
\text{global geometry}
\rightarrow
\text{local density}
\rightarrow
\text{fine detail}.
$$

This interpretation should not be imposed in advance unless required experimentally. One goal is to determine whether such specialization emerges naturally.

## Initial Experimental Design

The first study should avoid jointly learning every aspect of the decomposition. Otherwise, a negative result would be difficult to interpret.

A controlled initial experiment can proceed as follows:

1. Choose a family of intermediate distributions

   $$
   q_0,q_1,\ldots,q_M=p_{\mathrm{data}}.
   $$

2. Train a separate small stochastic bridge for every transition

   $$
   q_{i-1}\longrightarrow q_i.
   $$

3. Compare the composition with a monolithic model trained directly for

   $$
   q_0\longrightarrow p_{\mathrm{data}}.
   $$

4. Vary the number of modules, for example

   $$
   M\in\{1,2,4,8,16\}.
   $$

5. Match total inference FLOPs, total parameter count, or both.

6. Measure quality, numerical error, training stability, latency, and memory.

This produces a compositional scaling curve

$$
\operatorname{Quality}(M\mid B),
$$

where $B$ is a fixed total computational budget.

## Possible Outcomes

### Monolithic Regime

The best model uses

$$
M=1.
$$

This would indicate that shared representations across the complete stochastic process are more valuable than stage specialization.

Possible explanations include

- duplicated feature extraction across modules;
- inefficient parameter fragmentation;
- accumulated transition errors;
- insufficiently expressive intermediate interfaces;
- stronger benefits from sharing information across noise levels.

### Compositional Regime

The best model uses an intermediate number of modules:

$$
1<M^\star<\infty.
$$

This would suggest that there is a natural modular scale for learned stochastic transport.

Each module would receive a simpler task, while the total number of modules would remain small enough to avoid excessive error accumulation and computational overhead.

### Highly Modular Regime

Performance continues improving as the process is divided into increasingly small learned stochastic modules.

This would provide strong evidence that complex generative processes possess substantial compositional structure.

## Error-Propagation Perspective

Let $K_i^\star$ denote the ideal transition at stage $i$, and let $\widehat K_i$ denote the learned approximation.

Suppose the local approximation error is

$$
\varepsilon_i
=
\sup_q
W\left(
\widehat K_i q,
K_i^\star q
\right),
$$

where $W$ is an appropriate probability metric.

Under suitable stability assumptions, the final error may satisfy a bound of the form

$$
W\left(
\widehat K_M\cdots\widehat K_1q_0,
K_M^\star\cdots K_1^\star q_0
\right)
\leq
\sum_{i=1}^{M}
\left(
\prod_{j=i+1}^{M}L_j
\right)
\varepsilon_i,
$$

where $L_j$ measures the sensitivity of stage $j$ to errors in its input distribution.

This exposes the fundamental trade-off:

$$
\text{simpler local approximation problems}
\quad\text{versus}\quad
\text{accumulation and amplification of local errors}.
$$

The composition is beneficial when the reduction in local approximation difficulty is larger than the additional error introduced by composing multiple imperfect kernels.

## Relation to Learned Sources

The learned-source problem is a special case of this broader formulation.

A learned source followed by one SDE can be written as

$$
z
\xrightarrow{G_{\phi}}
q_1
\xrightarrow{K_{\theta}}
p_{\mathrm{data}}.
$$

The first stage performs generative computation outside the SDE, while the second performs stochastic refinement.

The multi-process formulation generalizes this to

$$
q_0
\xrightarrow{K_{\theta_1}}
q_1
\xrightarrow{K_{\theta_2}}
\cdots
\xrightarrow{K_{\theta_M}}
p_{\mathrm{data}}.
$$

The broader meta-question is therefore how generative computation should be factorized across multiple deterministic or stochastic modules.

## Scientific Value of a Negative Result

A negative result would not imply that the project failed.

If a carefully controlled comparison shows that one large stochastic model consistently dominates compositions of smaller models, this would provide evidence that complex learned stochastic processes are not efficiently decomposable under the tested conditions.

Such a result could reveal

- the importance of global representation sharing;
- the cost of repeatedly reconstructing features;
- the sensitivity of stochastic transport to intermediate approximation errors;
- limitations of modular probability-space representations;
- differences between compositionality in deterministic neural networks and stochastic generative processes.

The study should therefore be framed as an investigation rather than as an assumption that modularity must succeed.

## Main Contribution

The intended contribution is not merely a new multi-stage diffusion architecture.

The project studies the following general question:

> Do complex generative stochastic processes admit compute-efficient factorizations into compositions of low-capacity learned stochastic processes?

The objective is to characterize when

$$
\text{local specialization}
+
\text{compositional expressivity}
$$

outweigh

$$
\text{module overhead}
+
\text{loss of parameter sharing}
+
\text{accumulated approximation error}.
$$

## Concise Project Statement

We study whether a complex generative stochastic process can be represented more efficiently as a composition of genuinely small learned stochastic processes. Each component is implemented by a separate low-capacity score or drift model and learns a simpler distributional transition. Under matched parameter, inference-compute, and training-compute budgets, we compare these modular compositions with conventional monolithic stochastic generators. The goal is to determine whether learned stochastic processes exhibit compositional efficiency, identify the optimal module scale when it exists, and characterize the trade-off between local specialization and accumulated transition error. Both positive and negative results would provide evidence about the composability of complex generative processes.