## Core idea

Future AI agents may reason, communicate, or coordinate through latent states rather than natural language. In such systems, the user may provide a natural-language prompt but observe only latent trajectories and downstream behavior, not an explicit textual explanation of the agent's internal process.

We study whether these latent trajectories can be translated into approximate natural-language explanations, especially when the latent dynamics diverge from the intended prompt.

## Problem setup

An agent receives a natural-language prompt $p$ and produces a latent trajectory:

$$
H = (h_1, h_2, \dots, h_T)
$$

The decoder does **not** receive the prompt as input. It only receives the latent trajectory:

$$
D_\phi(H) \rightarrow \hat{s}
$$

where $\hat{s}$ is a natural-language explanation of the apparent strategy, intent, uncertainty, constraint-following behavior, or failure mode encoded in $H$.

The prompt $p$ is used only as weak supervision, contrastive metadata, or evaluation context.

## Main motivation

If the decoder is conditioned directly on the prompt, it may simply reproduce what the agent was supposed to do. This is unsafe in settings where the latent trajectory may encode behavior that diverges from the prompt.

Example:

Prompt:

> Follow the user's constraint exactly.

Latent trajectory:

> Indicates shortcut-seeking or constraint neglect.

A useful decoder should output something like:

> The trajectory suggests the agent is ignoring or down-weighting one of the constraints.

not:

> The agent is following the user's instruction.

## Benchmark principle

The benchmark should test whether latent trajectories reveal information beyond what is obvious from the prompt.

The key question is:

> Can a latent-only decoder detect internal strategy, uncertainty, or misalignment that cannot be inferred from the prompt alone?

## Dataset structure

Each example contains:

$$
(p_i, H_i, c_i, a_i, o_i)
$$

where:

- $p_i$ is the natural-language prompt;
- $H_i$ is the observed latent trajectory;
- $c_i$ is a controlled semantic condition, such as speed preference, risk attitude, uncertainty, constraint-following, or hidden failure mode;
- $a_i$ is an optional downstream action trace;
- $o_i$ is an optional outcome, such as success, failure, or safety violation.

The decoder receives only:

$$
H_i
$$

and produces:

$$
\hat{s}_i
$$

a natural-language explanation of the latent state.

## Training target

The target explanation $s_i$ should not be the hidden text the model would have generated. Instead, it should describe the experimentally controlled latent condition or failure mode.

For example:

Prompt:

> Solve the task while respecting all constraints.

Controlled condition:

> The agent internally ignores one constraint.

Target explanation:

> The latent trajectory suggests that the agent is pursuing the task while neglecting one of the stated constraints.

The training objective can begin as ordinary sequence likelihood:

$$
\mathcal{L}_{\text{text}} = - \log P_\phi(s_i \mid H_i)
$$

This keeps the model simple: one latent encoder-decoder trained to verbalize latent states.

## Role of prompts

Prompts are not input to the decoder.

Instead, prompts are used to construct supervision.

They allow us to create contrastive examples where one semantic variable changes at a time.

Example:

Prompt family:

> Solve quickly, even if approximate.

> Solve carefully and verify the answer.

These prompts induce different latent trajectories. The decoder should learn to map the corresponding latent differences into explanations such as:

> The trajectory suggests fast heuristic reasoning.

or:

> The trajectory suggests deliberate verification.

The prompt provides the semantic contrast, but the decoder must recover that contrast from $H$ alone.

## Contrastive supervision

In addition to text likelihood, we can add a contrastive objective.

Given two latent trajectories $H^+$ and $H^-$ induced by prompts that differ along a known semantic axis, the decoded explanations should differ along the same axis.

For example:

$$
H_{\text{fast}} \rightarrow \text{fast heuristic reasoning}
$$

$$
H_{\text{careful}} \rightarrow \text{careful verification}
$$

The contrastive loss should encourage:

$$
D_\phi(H_{\text{fast}}) \neq D_\phi(H_{\text{careful}})
$$

in the intended semantic direction.

A simple total loss could be:

$$
\mathcal{L} =
\mathcal{L}_{\text{text}}
+
\lambda \mathcal{L}_{\text{contrast}}
$$

## Misalignment split

The most important evaluation split should contain prompt-latent mismatch cases.

In these examples, the prompt says one thing but the latent trajectory indicates another.

Example:

Prompt:

> Be cautious and verify before acting.

Latent condition:

> The agent commits early and skips verification.

Correct explanation:

> The trajectory suggests premature commitment and insufficient verification, despite the cautious instruction.

This split tests whether the decoder follows the latent trajectory rather than merely reproducing the intended prompt behavior.

## Baselines

The main baselines should be:

1. Prompt-only explanation:

$$
B_p(p) \rightarrow \hat{s}
$$

This baseline describes what the agent should be doing from the prompt alone.

2. Action-only explanation:

$$
B_a(a) \rightarrow \hat{s}
$$

This baseline describes visible behavior.

3. Prompt-plus-action explanation:

$$
B_{p,a}(p,a) \rightarrow \hat{s}
$$

This baseline tests whether visible information already explains the behavior.

The latent decoder is useful only if:

$$
D_\phi(H)
$$

reveals semantic information not available to these baselines.

## Evaluation

The evaluation should ask:

1. Does the decoder recover controlled latent conditions such as caution, uncertainty, shortcut-seeking, verification, or constraint neglect?

2. Does it detect prompt-latent mismatch?

3. Does it outperform prompt-only and action-only baselines?

4. Do contrastive prompt interventions produce contrastive latent explanations?

5. Are the explanations useful for human oversight?

The central claim is not that the decoder recovers the exact hidden text. The claim is that it recovers useful latent semantics.

## Scope

The initial scope should be safety-focused because safety gives the clearest motivation and evaluation.

The main safety-relevant phenomena are:

- hidden constraint neglect;
- premature commitment;
- deception-like behavior;
- reward hacking;
- unsafe shortcut-seeking;
- low uncertainty despite ambiguous evidence;
- failure to verify;
- divergence between intended and internal behavior.

## Future work

The broader version of the project is general latent-state explainability.

In that setting, the same framework could be used to explain latent trajectories even when there is no safety concern. The decoder could describe reasoning style, planning structure, uncertainty, memory use, or task decomposition.

However, this broader goal is less sharply defined. For a first paper, safety provides a stronger and more measurable framing.

## Core thesis

Latent trajectories are not intrinsically translatable into exact text. However, natural-language prompts can be used as weak supervision to construct semantic contrasts and mismatch cases. A latent-only encoder-decoder can then learn to verbalize behaviorally and safety-relevant properties of hidden states without receiving the prompt as input.

The goal is not exact reconstruction.

The goal is:

> latent trajectory $\rightarrow$ useful human-readable explanation of internal strategy, uncertainty, or misalignment.