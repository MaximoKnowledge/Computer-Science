---
state: read
name: Interpreting Black Box Reward Models
link: https://alignment.openai.com/argo/
tldr: Distilled a natural language rubric from a black-box reward model
note:
quality:
  - good
abstract: "Reward models (RMs) have been the predominant mechanism in RLHF (Ziegler et al., 2019; Ouyang et al., 2022) for aligning models with user intent. Trained on millions of human preference judgments, they quietly shape what our models learn to do. This scale is powerful, but it comes with a cost: reward models can learn behaviors we do not fully understand.\r

  \r

  Reward models are trained on large, noisy, and heterogeneous datasets. As a result, they can pick up labeling artifacts and unintended biases, internalizing behaviors such as sycophancy or superficial stylistic preferences that may not reflect the behaviors we intended to encode (OpenAI et al., 2025).\r

  \r

  In contrast, LLM-as-a-judge methods (Zheng et al., 2023) rely on human designed rubrics: explicit, natural language criteria that specify what a good answer should look like. These rubrics are interpretable and steerable by construction, but only minimally calibrated against a small set of human preferences, unlike RMs that are trained on large-scale human data.\r

  \r

  This leads to a natural question: What do our reward models learn and what behaviors do they teach our models?"
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
  - safety
---
#paper
## Takeaways
- Defined rotation matrix R, looks to maximise projected kurtosis $z=U(Re_{i})$, where U is the unembedding matrix, R is the rotation matrix and $e_i$ is the i-th column of the weight matrix. The idea is to make the softmax of z have high kurtosis (i.e. be concentrated ~ low-entropy), while not losing information about $e_i$
- After training R, the most aligned tokens in the vocabulary space are the concepts $e_i$ is representing. To discover different concepts (for neurons that have superposition of concepts), they delete from the vocabulary previously discovered tokens

## I+D
-

## Deep Dive

