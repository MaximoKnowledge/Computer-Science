---
state: to-read
name: "The Spike, the Sparse and the Sink: Anatomy of Massive Activations and Attention Sinks"
link: https://arxiv.org/abs/2603.05498v1
tldr:
note:
quality:
abstract: "We study two recurring phenomena in Transformer language models: massive activations, in which a small number of tokens exhibit extreme outliers in a few channels, and attention sinks, in which certain tokens attract disproportionate attention mass regardless of semantic relevance. Prior work observes that these phenomena frequently co-occur and often involve the same tokens, but their functional roles and causal relationship remain unclear. Through systematic experiments, we show that the co-occurrence is largely an architectural artifact of modern Transformer design, and that the two phenomena serve related but distinct functions. Massive activations operate globally: they induce near-constant hidden representations that persist across layers, effectively functioning as implicit parameters of the model. Attention sinks operate locally: they modulate attention outputs across heads and bias individual heads toward short-range dependencies. We identify the pre-norm configuration as the key choice that enables the co-occurrence, and show that ablating it causes the two phenomena to decouple."
paper id: 2603.05498v1
authors:
  - Shangwen Sun
  - Alfredo Canziani
  - Yann LeCun
  - Jiachen Zhu
publication date: 2026-03-05T18:59
comments: ""
pdf: https://arxiv.org/pdf/2603.05498v1
tags: []
---
#paper
## Takeaways
- 

## I+D
- 

## Deep Dive

