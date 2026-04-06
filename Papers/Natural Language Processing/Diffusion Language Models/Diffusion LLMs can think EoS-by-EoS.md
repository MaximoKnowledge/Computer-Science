---
state: read
name: Diffusion LLMs can think EoS-by-EoS
link: https://arxiv.org/abs/2603.05197v1
tldr: DLMs use EoS to think. Really weak experiments
note:
quality:
  - meh
abstract: Diffusion LLMs have been proposed as an alternative to autoregressive LLMs, excelling especially at complex reasoning tasks with interdependent sub-goals. Curiously, this is particularly true if the generation length, i.e., the number of tokens the model has to output, is set to a much higher value than is required for providing the correct answer to the task, and the model pads its answer with end-of-sequence (EoS) tokens. We hypothesize that diffusion models think EoS-by-EoS, that is, they use the representations of EoS tokens as a hidden scratchpad, which allows them to solve harder reasoning problems. We experiment with the diffusion models LLaDA1.5, LLaDA2.0-mini, and Dream-v0 on the tasks Addition, Entity Tracking, and Sudoku. In a controlled prompting experiment, we confirm that adding EoS tokens improves the LLMs' reasoning capabilities. To further verify whether they serve as space for hidden computations, we patch the hidden states of the EoS tokens with those of a counterfactual generation, which frequently changes the generated output to the counterfactual. The success of the causal intervention underscores that the EoS tokens, which one may expect to be devoid of meaning, carry information on the problem to solve. The behavioral experiments and the causal interventions indicate that diffusion LLMs can indeed think EoS-by-EoS.
paper id: 2603.05197v1
authors:
  - Sarah Breckner
  - Sebastian Schuster
publication date: 2026-03-05T14:06
comments: ""
pdf: https://arxiv.org/pdf/2603.05197v1
tags:
  - talking-masks
  - dlms
  - xai
---
#paper
## Takeaways
- EoS tokens may carry useful information for DLMs' reasoning. Nonetheless, the experiments they conduct are pretty weak and limited (simple and short tasks)
- Experiments mostly break with LLaDA-2.0 mini, which they fail to explain
- They re-found that appending few EoS tokens to the ending of the sequence improves generation for free
- They patch activations of correct vs incorrect generations using the EoS tokens and observed that when inserting counterfactual EoS tokens, the generation degrades. Nonetheless they just do this for one step, raising noticeable concerns about their method; moreover, they don't provide any baseline (maybe patching the EoS with junk also degrades quality)

## I+D
- Can we scale this and actually prove it better? What happens on real NLP-reasoning datasets?

## Deep Dive

