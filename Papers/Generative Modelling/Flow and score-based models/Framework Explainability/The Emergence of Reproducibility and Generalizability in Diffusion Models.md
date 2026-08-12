---
state: skimmed
name: The Emergence of Reproducibility and Generalizability in Diffusion Models
link: https://arxiv.org/abs/2310.05264v5
tldr: Showed that different frameworks for training score models (DDPM, Score-based, Consistency Models, etc.) produce highly similar outputs when given the same input. Shedding some light into the generalisation of this models.
note:
quality:
  - banger
abstract: 'In this work, we investigate an intriguing and prevalent phenomenon of diffusion models which we term as "consistent model reproducibility": given the same starting noise input and a deterministic sampler, different diffusion models often yield remarkably similar outputs. We confirm this phenomenon through comprehensive experiments, implying that different diffusion models consistently reach the same data distribution and scoring function regardless of diffusion model frameworks, model architectures, or training procedures. More strikingly, our further investigation implies that diffusion models are learning distinct distributions affected by the training data size. This is supported by the fact that the model reproducibility manifests in two distinct training regimes: (i) "memorization regime", where the diffusion model overfits to the training data distribution, and (ii) "generalization regime", where the model learns the underlying data distribution. Our study also finds that this valuable property generalizes to many variants of diffusion models, including those for conditional use, solving inverse problems, and model fine-tuning. Finally, our work raises numerous intriguing theoretical questions for future investigation and highlights practical implications regarding training efficiency, model privacy, and the controlled generation of diffusion models.'
paper id: 2310.05264v5
authors:
  - Huijie Zhang
  - Jinfan Zhou
  - Yifu Lu
  - Minzhe Guo
  - Peng Wang
  - Liyue Shen
  - Qing Qu
publication date: 2023-10-08T19:02
comments: NeurIPS Diffusion Model Workshop 2023 (best paper award), the Forty-first International Conference on Machine Learning (ICML 2024)
pdf: https://arxiv.org/pdf/2310.05264v5
tags:
  - generative
  - sde
  - xai
---
#paper
## Takeaways
- Tested across various models with different losses: DDPM, SDE (Song score), EDM, Consistency Training, Consistency Distillation; architectures: DiT, UNet, EDM, ViT, etc; samplers and perturbation kernels. They found remarkably non-trivial reproducibility scores (introduced below).

The whole work is based on the reproducibility score, which is defined as the probability (so empirical ratio) that a given pair of images have an SSCD score of more than 0.6 (the SSCD score measures similarity of images):
$$\mathrm { R P \, S c o r e } : = \ { \mathbb P } \left( M _ { \mathrm { S S C D } } ( x _ { 1 }, x _ { 2 } ) > 0. 6 \right) = \frac{1}{N}\sum \mathbb{1}\{M_{SSCD(x_{1},x_{2})} \geq 0.6 \}$$
Where MSSCD comes from this computation of the SSCD (which is just the cosine similarity of the embeddings that SSCD produces between x1 and x2):
$$\mathcal { M } _ { \mathrm { S S C D } } ( x _ { 1 }, x _ { 2 } ) = \frac { \mathrm { S S C D } ( x _ { 1 } ) \cdot \mathrm { S S C D } ( x _ { 2 } ) } { \| \mathrm { S S C D } ( x _ { 1 } ) \| _ { 2 } \cdot \| \mathrm { S S C D } ( x _ { 2 } ) \| _ { 2 } }$$
SSCD is a model, just like DINO or any other image-trained model. So what they do on practice is sample 10k seeds (N=10k) generate images with each model (get the x for each model), and use them as input for SSCD, with which we then compute the probability of having a cosine greater than 0.6. 
Another metric they introduce is the generalisation score:
$$ \mathrm { G L ~ S c o r e } : = 1 - \mathbb { P } \left( \operatorname* { m a x } _ { i \in [ N ] } \left[ \mathcal { M } _ { \mathrm { S S C D } } ( x, y _ { i } ) \right] > 0. 6 \right)$$
This is the same thing but done with the dataset. So we try to measure how close the generated images resemble the closest ones seen at training time (that's why we take the max). This is a summary of their most impactful result:
![[Pasted image 20260806192604.png|320]]
This implies that models trained with different things, all tend to answer to noise in similar ways, as the resulting images have similarities higher than 0.6 in the proportion shown. Another interesting result is that the manifolds seem to be similarly-shaped:
![[Pasted image 20260806192801.png]]
Taking three noise samples (e1,e2,e3) and producing images gives us those images for each model (this follows from the previous plot). But here they construct a 2D hyperplane
$$\epsilon \left( \alpha, \beta \right) = \alpha \cdot \left( \epsilon _ { 2 } - \epsilon _ { 1 } \right) + \beta \cdot \left( \epsilon _ { 3 } - \epsilon _ { 1 } \right) + \epsilon _ { 1 }$$
Where they make a 100x100 sweep grid over the values of $\alpha$ and $\beta$. They then plot:
$$\max_{i} M_{\text{SSCD}}(x_{i}, x(\alpha, \beta)) $$
$x(\alpha, \beta)$ is the sample generated from the corresponding interpolated noise. They finally assign to this score a colour depending on which of the three images is closer (so the three vertical lines correspond to the MSSCD score of the generated image w.r.t. that denoised noise e_i). Note how the hyperplanes are pretty similar, which implies that the models induce similar perceptual partitions on a sampled two-dimensional slice of noise space. Moreover, the smoothness of the transitions sheds some light into the Lipschitzness of the networks.

- They finally briefly show how reproducibility covaries positively with generalisation of the model

## I+D
- They didn't study representations (though this is an old paper). As everything seems to be converging, are representations also converging, or these models build different mechanisms to represent the same underlying "score"

## Deep Dive

