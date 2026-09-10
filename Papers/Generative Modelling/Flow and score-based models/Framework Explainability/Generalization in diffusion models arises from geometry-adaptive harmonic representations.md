---
state: read
name: Generalization in diffusion models arises from geometry-adaptive harmonic representations
link: https://arxiv.org/abs/2310.02557v3
tldr: Analysed diffusion models through the Jacobian's eigenvectors and values to find inductive biases of these models. Also showed that these models converge to the same score function even when trained with different data
note:
quality:
  - banger
abstract: Deep neural networks (DNNs) trained for image denoising are able to generate high-quality samples with score-based reverse diffusion algorithms. These impressive capabilities seem to imply an escape from the curse of dimensionality, but recent reports of memorization of the training set raise the question of whether these networks are learning the "true" continuous density of the data. Here, we show that two DNNs trained on non-overlapping subsets of a dataset learn nearly the same score function, and thus the same density, when the number of training images is large enough. In this regime of strong generalization, diffusion-generated images are distinct from the training set, and are of high visual quality, suggesting that the inductive biases of the DNNs are well-aligned with the data density. We analyze the learned denoising functions and show that the inductive biases give rise to a shrinkage operation in a basis adapted to the underlying image. Examination of these bases reveals oscillating harmonic structures along contours and in homogeneous regions. We demonstrate that trained denoisers are inductively biased towards these geometry-adaptive harmonic bases since they arise not only when the network is trained on photographic images, but also when it is trained on image classes supported on low-dimensional manifolds for which the harmonic basis is suboptimal. Finally, we show that when trained on regular image classes for which the optimal basis is known to be geometry-adaptive and harmonic, the denoising performance of the networks is near-optimal.
paper id: 2310.02557v3
authors:
  - Zahra Kadkhodaie
  - Florentin Guth
  - Eero P. Simoncelli
  - Stéphane Mallat
publication date: 2023-10-04T03:30
comments: Accepted for oral presentation at ICLR, Vienna, May 2024
pdf: https://arxiv.org/pdf/2310.02557v3
tags:
  - generative
  - xai
  - twisted-maps
---
#paper
## Takeaways
- Trained diffusion models on non-overlapping subsets of CelebA, and from the same initial noise sample (so almost Gaussian) the two models generate remarkably similar images. As the size of the training set is reduced for each model, they both tend more to memorise and just denoise/generate towards faces inside the dataset. Finally, showed that cosine similarity between two generative model samples increases as sample size increases, while the cosine with the closest training dataset image decreases (thus, again, more data means less memorisation and more convergence to a shared solution; not necessarily proof of convergence to the true distribution, but at least convergence to the same learned solution).

* By analysing the Jacobian of networks without biases (literally without the sum of a constant), one can show many properties of the eigenvalues and eigenvectors that help analyse the network. The bias-free part is what gives the nice equality below, because the network is piecewise linear rather than piecewise affine. Separately, for the eigendecomposition, they assume the Jacobian is symmetric positive semi-definite; they show this exactly for the optimal denoiser and use it as a good empirical approximation for the trained networks. The whole work spans from this equality:
  $$f(y) = \nabla f(y),y = \sum_k \lambda_k(y),\langle y,e_k(y)\rangle,e_k(y)$$
  Where $\nabla f(y)$ is the Jacobian of the network at the input $y$. Note that the eigenvectors $e_k(\cdot)$ and eigenvalues $\lambda_k(\cdot)$ are functions of the noisy image $y$ (so they're not constant). Also, these particular networks are blind/universal denoisers, meaning they do not receive the timestep/noise level as input; this is why there is no explicit $\sigma$ or $t$ in the expression. This is separate from the “bias-free” assumption. Also, the whole paper analyses diffusion models through the lens of denoisers, so by making an equivalence between predicting the score function, and predicting the expected clean image based on a noisy input.

* They show that the denoising objective can be understood through a trade-off between the Jacobian’s effective dimensionality and the accuracy when predicting the clean image: $$\operatorname{MSE}(f,\sigma^2)
  \mathbb{E}_{y}
  \left[
  2\sigma^2 \operatorname{tr}\nabla f(y)
  +
  \lVert y-f(y)\rVert^2
  \sigma^2 d
  \right]$$So, the model needs to make the trace of the Jacobian as small as possible (roughly: keep few/highly relevant directions active, or have many eigenvalues close to 0), but also match the image accurately. So these two things: ignoring as many directions as possible, and reconstruction; compete at the optimum. This is not exactly “rank” in the strict linear algebra sense, but it behaves like a soft/effective dimensionality measure.

* Showed that diffusion denoisers have an inductive bias toward Jacobian eigenvectors whose pixel patterns follow the shapes, regions and boundaries of the input image. This lets them preserve structured variations while removing pixel changes that do not fit the image’s geometry. This structure is what they call GAHBs (geometry-aware harmonic basis). So the network behaves as a denoiser in the signal-processing sense: it filters some eigenvectors (the ones with low eigenvalues, which are directions the denoiser is relatively insensitive to around that input) and preserves others (the ones with high eigenvalues, which represent locally plausible data/image directions).

* The reason GAHBs matter in the paper is that they are supposed to explain generalisation. The premise is roughly: the model does not need to memorise every training image because it learns this geometry-aware basis/filtering structure, and that structure transfers across images. So instead of learning arbitrary high-dimensional image densities from scratch, the convolutional denoiser is biased toward a useful class of shape-following representations. This is not 100% proven as a full generalisation theorem; it is more the direction the evidence points to. The CelebA experiments show reduced memorisation and convergence to a shared solution, while the toy examples and controlled datasets support the idea that GAHBs are a real architectural/denoising bias and not just a pretty visualization.


## I+D
- The Celeba way of demonstrating true convergence is really good and applicable broadly to show convergence of models to a shared solution
- The GAHB thing seems mostly correlational (i.e. the model may be converging to that as a side-effect of something else), not 100% isolated from any other possible phenomenon that may relate to it. Though it's pretty well presented

## Deep Dive

