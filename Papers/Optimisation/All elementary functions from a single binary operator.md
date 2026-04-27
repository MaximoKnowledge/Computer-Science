---
state: skimmed
name: All elementary functions from a single binary operator
link: https://arxiv.org/abs/2603.21852v2
tldr: With the double input function eml(x,y)=exp(x)-ln(y), you can represent any function in a scientific calculator
note:
quality:
  - banger
abstract: "A single two-input gate suffices for all of Boolean logic in digital hardware. No comparable primitive has been known for continuous mathematics: computing elementary functions such as sin, cos, sqrt, and log has always required multiple distinct operations. Here I show that a single binary operator, eml(x,y)=exp(x)-ln(y), together with the constant 1, generates the standard repertoire of a scientific calculator. This includes constants such as e, pi, and i; arithmetic operations including addition, subtraction, multiplication, division, and exponentiation as well as the usual transcendental and algebraic functions. For example, exp(x)=eml(x,1), ln(x)=eml(1,eml(eml(1,x),1)), and likewise for all other operations. That such an operator exists was not anticipated; I found it by systematic exhaustive search and established constructively that it suffices for the concrete scientific-calculator basis. In EML (Exp-Minus-Log) form, every such expression becomes a binary tree of identical nodes, yielding a grammar as simple as S -> 1 | eml(S,S). This uniform structure also enables gradient-based symbolic regression: using EML trees as trainable circuits with standard optimizers (Adam), I demonstrate the feasibility of exact recovery of closed-form elementary functions from numerical data at shallow tree depths up to 4. The same architecture can fit arbitrary data, but when the generating law is elementary, it may recover the exact formula."
paper id: 2603.21852v2
authors:
  - Andrzej Odrzywołek
publication date: 2026-03-23T11:40
comments: 2 figures, Supplementary Information, code available at https://zenodo.org/records/19183008
pdf: https://arxiv.org/pdf/2603.21852v2
tags: []
---
#paper
## Takeaways
-

## I+D
- Is this the new activation function killer?

## Deep Dive

