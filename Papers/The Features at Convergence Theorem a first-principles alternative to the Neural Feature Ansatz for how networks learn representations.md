---
state: skimmed
name: "The Features at Convergence Theorem: a first-principles alternative to the Neural Feature Ansatz for how networks learn representations"
link: https://arxiv.org/abs/2507.05644v2
tldr: Proposed a convergence equality of a matrix's product with itself, which essentially would say what the weight matrices should satisfy once they converge to a local stationary point
note:
quality:
  - good
abstract: It is a central challenge in deep learning to understand how neural networks learn representations. A leading approach is the Neural Feature Ansatz (NFA) (Radhakrishnan et al. 2024), a conjectured mechanism for how feature learning occurs. Although the NFA is empirically validated, it is an educated guess and lacks a theoretical basis, and thus it is unclear when it might fail, and how to improve it. In this paper, we take a first-principles approach to understanding why this observation holds, and when it does not. We use first-order optimality conditions to derive the Features at Convergence Theorem (FACT), an alternative to the NFA that (a) obtains greater agreement with learned features at convergence, (b) explains why the NFA holds in most settings, and (c) captures essential feature learning phenomena in neural networks such as grokking behavior in modular arithmetic and phase transitions in learning sparse parities, similarly to the NFA. Thus, our results unify theoretical first-order optimality analyses of neural networks with the empirically-driven NFA literature, and provide a principled alternative that provably and empirically holds at convergence.
paper id: 2507.05644v2
authors:
  - Enric Boix-Adsera
  - Neil Mallinar
  - James B. Simon
  - Mikhail Belkin
publication date: 2025-07-08T03:52
comments: ""
pdf: https://arxiv.org/pdf/2507.05644v2
tags:
  - dl
  - theory
---
#paper
## Takeaways
- They propose FACT, which is this equality (the sum is over training examples):
$$W ^ { \top } W = \mathsf { F A C T } : = - \frac { 1 } { n \lambda } \sum _ { i = 1 } ^ { n } ( \nabla _ { h } \ell _ { i } ) ( h ( x _ { i } ) ) ^ { \top } \,$$
Where W is the weight matrix being analysed and h and l are defined as follows:
$$f ( x ; \theta ) = g ( W h ( x ), x )$$
$$\nabla _ { h } \ell _ { i } : = \frac { \partial \ell ( g ( W h, x ) ; y _ { i } ) } { \partial h } \mid _ { h = h ( x _ { i } ) } \in \mathbb { R } ^ { d } \qquad \nabla _ { h } f _ { i } : = \frac { \partial g ( W h, x ) } { \partial h } \mid _ { h = h ( x _ { i } ) } \in \mathbb { R } ^ { d \times c }$$
Essentially what they do is isolate the weight matrix from the rest of the network and observe its partial derivatives (note that as the loss is a scalar we get a vector, and the partial of g w.r.t. g is a matrix as g is the rest of the NN given x and the output of the matrix multiplied by h). So, what the framework buys is a local stationary guarantee of what the matrix W multiplied by its transpose converges to, which is useful when analysing learning dynamics.

## I+D
-

## Deep Dive

