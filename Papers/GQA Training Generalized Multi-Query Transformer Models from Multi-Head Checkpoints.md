---
state: read
name: "GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints"
link: https://arxiv.org/abs/2305.13245v3
tldr: Introduced grouped-query attention
note:
quality:
  - banger
abstract: Multi-query attention (MQA), which only uses a single key-value head, drastically speeds up decoder inference. However, MQA can lead to quality degradation, and moreover it may not be desirable to train a separate model just for faster inference. We (1) propose a recipe for uptraining existing multi-head language model checkpoints into models with MQA using 5% of original pre-training compute, and (2) introduce grouped-query attention (GQA), a generalization of multi-query attention which uses an intermediate (more than one, less than number of query heads) number of key-value heads. We show that uptrained GQA achieves quality close to multi-head attention with comparable speed to MQA.
paper id: 2305.13245v3
authors:
  - Joshua Ainslie
  - James Lee-Thorp
  - Michiel de Jong
  - Yury Zemlyanskiy
  - Federico Lebrón
  - Sumit Sanghai
publication date: 2023-05-22T17:16
comments: Accepted at EMNLP 2023. Added to related work
pdf: https://arxiv.org/pdf/2305.13245v3
tags:
  - architecture
  - llm
  - efficiency
---
#paper
## Takeaways
- GQA (Grouped-query attention) can be summarised in one image
![[Pasted image 20261003084356.png]]
Essentially, GQA keeps a $W_K$ and $W_V$ matrices fixed for each group consisting of $k$ different $W_Q$ heads. The idea is a trade-off between MHA flexibility but expensive computation and MQA super efficient but reductionist approach.
## I+D
-

## Deep Dive

