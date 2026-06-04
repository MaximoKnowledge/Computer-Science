---
state: to-read
name: "NITP: Next Implicit Token Prediction for LLM Pre-training"
link: https://arxiv.org/abs/2605.24956v1
tldr: Introduced Deep Supervision for consistency in pre-training representation of tokens, arguing that it avoids the model overfitting the one-hot representations and creating better concept embeddings
note:
quality:
abstract: Standard next-token prediction (NTP) supervises language models solely through discrete labels in the output logit space. We argue that this sparse one-hot supervision leaves the latent representation space under-constrained, allowing hidden states to drift into degenerate and anisotropic configurations that can limit generalization. To address this issue, we propose Next Implicit Token Prediction (NITP), which augments discrete prediction with dense continuous supervision directly in the representation space. NITP trains the model to predict the implicit semantic content of the next token, using shallow-layer representations from the same model as stable self-supervised targets. We provide theoretical analysis showing that NITP regularizes the optimization landscape by mitigating under-constrained degrees of freedom and encouraging a compact, structured representation geometry. Empirically, across dense and MoE models ranging from 0.5B to 9B parameters, NITP consistently improves downstream performance with negligible computational overhead. On a 9B MoE model, NITP achieves a 5.7% absolute improvement on MMLU-Pro, along with gains of 6.4% on C3 and 4.3% on CommonsenseQA, with approximately 2% additional training FLOPs and no additional inference cost. Our implementation is available at https://github.com/aHapBean/NITP.
paper id: 2605.24956v1
authors:
  - Xiangdong Zhang
  - Debing Zhang
  - Shaofeng Zhang
  - Xiaohan Qin
  - Yu Cheng
  - Junchi Yan
publication date: 2026-05-24T09:13
comments: Accepted at ICML 2026
pdf: https://arxiv.org/pdf/2605.24956v1
tags:
  - llm
  - nlp
  - ul
---
#paper
## Takeaways
-

## I+D
-

## Deep Dive

