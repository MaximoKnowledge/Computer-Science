---
state: to-read
name: "When Models Manipulate Manifolds: The Geometry of a Counting Task"
link: https://arxiv.org/abs/2601.04480v1
tldr: Defined through various experiments how Claude Haiku represents character count. Rigorously characterised the spanned subspace
note:
quality:
  - good
abstract: "Language models can perceive visual properties of text despite receiving only sequences of tokens-we mechanistically investigate how Claude 3.5 Haiku accomplishes one such task: linebreaking in fixed-width text. We find that character counts are represented on low-dimensional curved manifolds discretized by sparse feature families, analogous to biological place cells. Accurate predictions emerge from a sequence of geometric transformations: token lengths are accumulated into character count manifolds, attention heads twist these manifolds to estimate distance to the line boundary, and the decision to break the line is enabled by arranging estimates orthogonally to create a linear decision boundary. We validate our findings through causal interventions and discover visual illusions--character sequences that hijack the counting mechanism. Our work demonstrates the rich sensory processing of early layers, the intricacy of attention algorithms, and the importance of combining feature-based and geometric views of interpretability."
paper id: 2601.04480v1
authors:
  - Wes Gurnee
  - Emmanuel Ameisen
  - Isaac Kauvar
  - Julius Tarng
  - Adam Pearce
  - Chris Olah
  - Joshua Batson
publication date: 2026-01-08T01:33
comments: ""
pdf: https://arxiv.org/pdf/2601.04480v1
tags:
  - llm
  - xai
  - talking-masks
---
#paper
## Takeaways
- Trained huge 10 million sparse crosscoder (~= SAE). Found 10 features (among the 10M) firing at different quantities of line length count (i.e. how many characters before the current token)
- Did PCA on the average vector for each line count, i.e. for each token in their dataset, they assigned a label that is the number of characters since the last \n; then they did the average of the hidden states for each number of characters (so they got 150 averages) and performed PCA on that. With 6D they explained 95% of the variance, meaning that the character count is low-dimensional. They plotted two 3D plots and got interesting hellixes as the character count increases
- Extremely simple and effective patching. With the dataset of 150 mean vectors they subtract the mean from the original line count and add the one from the modified one: $a_{\text{patched}}=a_{\text{original}}-\mu_{\text{original}}+\mu_{\text{target}}$. They find that this causal patching makes the model change where it predicts the newlines, proving the causality of the signal
- They trained 150 logistic regression probes to predict each number of character counts. And they found a ringed receptor field, i.e. the logistic regressions fire in a "cyclical" way
![[Pasted image 20260503112318.png]]
- The other experiments rely heavily on the attribution graph and crosscoder, so they're less replicable

## I+D
-

## Deep Dive

