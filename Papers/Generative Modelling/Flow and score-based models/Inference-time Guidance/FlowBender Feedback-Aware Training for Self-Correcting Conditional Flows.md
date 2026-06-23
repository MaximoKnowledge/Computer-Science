---
state: skimmed
name: "FlowBender: Feedback-Aware Training for Self-Correcting Conditional Flows"
link: https://arxiv.org/abs/2606.20404v1
tldr: Proposed closed-loop feedback for flow-models guidance. Effectively acting as a CFG guided by some error signal available at inference
note:
quality:
  - good
abstract: "Conditional diffusion and flow models routinely fail to satisfy the very constraints that define their task. For instance, a depth-conditioned model often produces images whose re-extracted depth disagrees with the input, even though the forward operator--the depth predictor defining the constraint--is available during both training and inference. Existing approaches generally fall into two categories: supervised models that treat the conditioning signal as a static cue and ignore alignment information at inference, and guidance-based methods that consult it through hand-tuned linear updates, typically trading fidelity to the condition against the plausibility of the generated sample. We argue that the fundamental gap in both paradigms is that the model is never trained to utilize its own alignment error. We introduce FlowBender, a closed-loop framework that treats this error as a first-class input, training the network to learn a correction policy conditioned on inference-time feedback. At each step, an unguided look-ahead pass estimates the clean signal, a task-specific deviation is computed via the forward operator, and a refinement pass consumes this signal to produce a corrected velocity. We propose several variants of FlowBender, including a gradient-based formulation for differentiable operators and a zero-order variant for non-differentiable settings such as JPEG compression. For efficient sampling, we introduce a prior-step shortcut that enables closed-loop correction at a minimal additional computational cost. Across image-to-image translation, restoration, and 3D mesh texturing, FlowBender consistently outperforms standard supervised baselines, alignment-loss-augmented training, and state-of-the-art inference-time guidance, improving fidelity and plausibility simultaneously rather than trading them against each other. Project page: https://flow-bender.github.io/"
paper id: 2606.20404v1
authors:
  - Daniel Gilo
  - Sven Elflein
  - Ido Sobol
  - Or Litany
publication date: 2026-06-18T15:56
comments: "Project page: https://flow-bender.github.io/"
pdf: https://arxiv.org/pdf/2606.20404v1
tags:
  - generative
  - flow
---
#paper
## Takeaways
- Made flow-models have as input a feedback signal computed from an available error function at inference. Works as an inference-time alignment method
- During training 2 NFEs are done for each step: 1 with the unconditional model that makes a first guess, and one that takes the feedback from the just-mentioned inference and uses it to compute the optimisation loss (note that there's a stop-grad to avoid optimising the unconditional pass, as it's used as a reference and not meant to be optimised, refer to the paper for a cleaner image)
- Proposed both a zero and first-order corrector: zero uses just a difference in a metric, while first computes the gradient. These signals are the ones that the model then retakes as feedback input
- At inference defined a maximum-time threshold, above which the feedback loop is just re-used with the previous feedback-corrected prediction (i.e. we don't do 2 NFEs for each step but use the previous prediction to compute the feedback)

## I+D
- It's essentially really similar to other reward-alignment techniques in some sense, one could use a closed-loop system to guide at inference time the model towards better rewards. Nonetheless their whole framework is built upon the assumption of having access to an operator $\mathcal{H}$ which can evaluate latents and not just final images. Therefore, for reward-adaptation some more machinery is done, as a one-step predictor would be used in the middle

## Deep Dive

