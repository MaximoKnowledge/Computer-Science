---
state: skimmed
name: Disentangling MLP Neuron Weights in Vocabulary Space
link: https://arxiv.org/abs/2604.06005v1
tldr: Extracted concepts from the weight matrices of MLPs by rotating the columns of the matrices and maximising kurtosis
note:
quality:
  - good
abstract: "Interpreting the information encoded in model weights remains a fundamental challenge in mechanistic interpretability. In this work, we introduce ROTATE (Rotation-Optimized Token Alignment in weighT spacE), a data-free method requiring no forward passes that disentangles MLP neurons directly in weight space. Our approach relies on a key statistical observation: neurons that encode coherent, monosemantic concepts exhibit high kurtosis when projected onto the model's vocabulary. By optimizing rotations of neuron weights to maximize their vocabulary-space kurtosis, our method recovers sparse, interpretable directions which we name vocabulary channels. Experiments on Llama-3.1-8B-Instruct and Gemma-2-2B-it demonstrate that ROTATE consistently recovers vocabulary channels that are faithful to the neuron's behavior. ablating individual channels selectively disables corresponding input activations or the promotion of specific concepts. Moreover, aggregating channel-level descriptions yields comprehensive neuron descriptions that outperform optimized activation-based baselines by 2-3x in head-to-head comparisons. By providing a data-free decomposition of neuron weights, ROTATE offers a scalable, fine-grained building block for interpreting LMs."
paper id: 2604.06005v1
authors:
  - Asaf Avrahamy
  - Yoav Gur-Arieh
  - Mor Geva
publication date: 2026-04-07T15:39
comments: ""
pdf: https://arxiv.org/pdf/2604.06005v1
tags:
  - llm
  - xai
---
#paper
## Takeaways
- Defined rotation matrix R, looks to maximise projected kurtosis $z=U(Re_{i})$, where U is the unembedding matrix, R is the rotation matrix and $e_i$ is the i-th column of the weight matrix. The idea is to make the softmax of z have high kurtosis (i.e. be concentrated ~ low-entropy), while not losing information about $e_i$
- After training R, the most aligned tokens in the vocabulary space are the concepts $e_i$ is representing. To discover different concepts (for neurons that have superposition of concepts), they delete from the vocabulary previously discovered tokens

## I+D
-

## Deep Dive

