---
state: to-read
name: Diffusion Language Models Are Natively Length-Aware
link: https://arxiv.org/abs/2603.06123v1
tldr: Computed the cumulative probability of EOS not being at position i and used it to crop the sequence when this cumulative probability drops below a threshold
note:
quality:
  - good
abstract: Unlike autoregressive language models, which terminate variable-length generation upon predicting an End-of-Sequence (EoS) token, Diffusion Language Models (DLMs) operate over a fixed maximum-length context window for a predetermined number of denoising steps. However, this process is independent of the required response length, resulting in computational waste for the majority of short responses common in reasoning and chat tasks. To address this problem, we conjecture that the latent prompt representation contains sufficient information to estimate the required output length. We provide empirical evidence for this phenomenon and propose a zero-shot mechanism to dynamically crop the context window before generation begins, leading to fewer diffusion steps and substantial computational savings. We evaluate our approach on four benchmarks with diverse tasks -- GSM8K (reasoning), HumanEval (code generation), IfEval (instruction following), and LongFormQA (question answering) -- revealing massive efficiency gains at minimal performance impact. We report significant reductions in FLOPs across all tasks, with no statistically significant performance degradation, and significant performance improvements in 2 out of 4 tasks.
paper id: 2603.06123v1
authors:
  - Vittorio Rossi
  - Giacomo Cirò
  - Davide Beltrame
  - Luca Gandolfi
  - Paul Röttger
  - Dirk Hovy
publication date: 2026-03-06T10:30
comments: ""
pdf: https://arxiv.org/pdf/2603.06123v1
tags:
  - dlms
  - xai
---
#paper
## Takeaways
- Computed for each token the probability of having an EOS, then did the cumulative product of the complement (1-P(EOS)) in token order to compute the probability that no EOS has happened yet. So compute the cumulative product of complements and take the complement of that (it's the % that no EOS has happened before position i, i.e. the sequence finished before). If it's bigger than an epsilon (e.g. 0.95), the remaining tokens are cut
- Also found that the best performance at IfEval was using this method, and not adding/subtracting more MASK tokens (though subtracting more was not so detrimental as adding). Weak last experiment
- Only tested on LLaDA

## I+D
-

## Deep Dive

