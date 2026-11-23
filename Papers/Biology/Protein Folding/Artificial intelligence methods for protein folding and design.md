---
state: read
name: Artificial intelligence methods for protein folding and design
link: https://pubmed.ncbi.nlm.nih.gov/40505455/
tldr: Survey of the 2024 state of protein prediction. From structure design to inverse prediction (from shape to aminoacid)
note:
quality:
  - banger
abstract: Machine learning has revolutionized protein structure prediction and design. This review discusses current methods for protein folding and inverse folding challenges. Models like AlphaFold2 (AF2), RoseTTAFold, and ESMFold excel at leveraging evolutionary information to accurately predict protein structures while still struggling to capture the physics of protein folding. Their repurposing for protein design has led to innovations such as RFdiffusion, AF2-design, and relaxed sequence optimization. ProteinMPNN and ESM-IF design sequences based on structure, so they are frequently referred to as "inverse folding' methods. By examining the potential and limitations of current protein design methods and metrics, we provide perspectives on developing models that fully characterize energy landscapes associated with amino acid sequences. Such advances would enable more accurate structure prediction and the design of proteins with specified conformational dynamics, potentially transforming our ability to engineer novel proteins for biotechnological applications.
paper id: 1606.02185v2
authors:
  - Harrison Edwards
  - Amos Storkey
publication date: 2025-06-11T15:36:00
comments: Updated to camera ready version for ICLR 2017
pdf: https://arxiv.org/pdf/1606.02185v2
tags:
  - generative
  - bio
  - survey
  - limitations
---
#paper
## Takeaways
- Although models are able to predict really good in-distribution protein sequences, as things get harder and more realistic, they struggle to do so. Experimental validation seems limited, and many researchers claim that these models don't recover the underlying energy function that proteins follow
- Models seem to just learn local patterns where aminoacids close in the sequence are close in 3D. While nature has shown that's not always the case
- Using AF2 as the validator is not the best guarantee, as inconsistencies continue to arise

## I+D
-

## Deep Dive

