---
state: skimmed
name: Stochastic Parameter Decomposition
link: https://arxiv.org/abs/2506.20790v2
tldr: Decomposed weight matrices into rank-one matrices so that components can be attributed to minimal ranks producing them. Achieved this through 4 losses enforcing some conditions
note:
quality:
abstract: A key step in reverse engineering neural networks is to decompose them into simpler parts that can be studied in relative isolation. Linear parameter decomposition -- a framework that has been proposed to resolve several issues with current decomposition methods -- decomposes neural network parameters into a sum of sparsely used vectors in parameter space. However, the current main method in this framework, Attribution-based Parameter Decomposition (APD), is impractical on account of its computational cost and sensitivity to hyperparameters. In this work, we introduce \textit{Stochastic Parameter Decomposition} (SPD), a method that is more scalable and robust to hyperparameters than APD, which we demonstrate by decomposing models that are slightly larger and more complex than was possible to decompose with APD. We also show that SPD avoids other issues, such as shrinkage of the learned parameters, and better identifies ground truth mechanisms in toy models. By bridging causal mediation analysis and network decomposition methods, this demonstration opens up new research possibilities in mechanistic interpretability by removing barriers to scaling linear parameter decomposition methods to larger models. We release a library for running SPD and reproducing our experiments at https://github.com/goodfire-ai/spd/tree/spd-paper.
paper id: 2506.20790v2
authors:
  - Lucius Bushnaq
  - Dan Braun
  - Lee Sharkey
publication date: 2025-06-25T19:26
comments: ""
pdf: https://arxiv.org/pdf/2506.20790v2
tags: []
---
#paper
## Takeaways
- Similarly to SVD (but not exactly equal as they don't do SVD), they decompose a matrix into C rank-one components (note that biases can be embedded as a column of $W$): $$ W_{i,j} \approx \sum^C_{c=1}U_{i,c} V_{c,j}$$
- From those proposed matrices they optimise several different metrics to enforce different qualities.
**Faithfulness** (how close is the reconstructed matrix): $\frac{1}{N}\sum^L \sum_{i,j} (W_{i,j}-\sum^C_{c=1}U_{i,c} V_{c,j})^2$
**Causal importance**
**c**

## I+D
-

## Deep Dive

