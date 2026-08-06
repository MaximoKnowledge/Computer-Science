---
state: read
name: Simplifying, Stabilizing and Scaling Continuous-Time Consistency Models
link: https://arxiv.org/abs/2410.11081v2
tldr: Proposed an alternative way of training continuous-time CMs which greatly improves training stability
note:
quality:
  - good
abstract: Consistency models (CMs) are a powerful class of diffusion-based generative models optimized for fast sampling. Most existing CMs are trained using discretized timesteps, which introduce additional hyperparameters and are prone to discretization errors. While continuous-time formulations can mitigate these issues, their success has been limited by training instability. To address this, we propose a simplified theoretical framework that unifies previous parameterizations of diffusion models and CMs, identifying the root causes of instability. Based on this analysis, we introduce key improvements in diffusion process parameterization, network architecture, and training objectives. These changes enable us to train continuous-time CMs at an unprecedented scale, reaching 1.5B parameters on ImageNet 512x512. Our proposed training algorithm, using only two sampling steps, achieves FID scores of 2.06 on CIFAR-10, 1.48 on ImageNet 64x64, and 1.88 on ImageNet 512x512, narrowing the gap in FID scores with the best existing diffusion models to within 10%.
paper id: 2410.11081v2
authors:
  - Cheng Lu
  - Yang Song
publication date: 2024-10-14T20:43
comments: ICLR 2025 Oral
pdf: https://arxiv.org/pdf/2410.11081v2
tags:
  - generative
  - sde
  - twisted-maps
---
#paper
## Takeaways
For an introduction to Consistency Models see [Consistency Models Introduction](obsidian://open?vault=Computer%20Science&file=Machine%20Learning%2FGenerative%20Models%2FConsistency%20Models%2FConsistency%20Models%20Introduction)

* They introduce **TrigFlow**, which gives data and noise the same variance and mixes them through a rotation:
$$
  x_t=\cos(t)x_0+\sin(t)z,
  \qquad
  z\sim\mathcal N(0,\sigma_d^2I),
  $$ $$
  v_t=\frac{\mathrm dx_t}{\mathrm dt}
  =\cos(t)z-\sin(t)x_0,
  \qquad
  t\in[0,\pi/2].
  $$
* Using TrigFlow the trajectory velocity becomes:  $$
  \frac{\mathrm dx_t}{\mathrm dt}
  =
  \sigma_dF_\theta\!\left(\frac{x_t}{\sigma_d},c_{\mathrm{noise}}(t)\right),
  $$$$
  \mathcal L_{\mathrm{Diff}}
  =
  \mathbb E\left[
  \left\|
  \sigma_dF_\theta\!\left(\frac{x_t}{\sigma_d},c_{\mathrm{noise}}(t)\right)-v_t
  \right\|_2^2
  \right].
  $$* The consistency model is parameterized as a one-step first-order solution of this ODE:  $$
  f_\theta(x_t,t)
  =
  \cos(t)x_t-\sin(t)\sigma_d
  F_\theta\!\left(\frac{x_t}{\sigma_d},c_{\mathrm{noise}}(t)\right).
  $$  Therefore $f_\theta(x,0)=x$ automatically, rather than requiring the boundary condition to be learned.
* Under this parameterization, the tangent used for continuous consistency training is  $$
  \frac{\mathrm df_{\theta^-}}{\mathrm dt}
  =
  -\cos(t)\left(\sigma_dF_{\theta^-}-\frac{\mathrm dx_t}{\mathrm dt}\right)
  -\sin(t)\left(x_t+\sigma_d\frac{\mathrm dF_{\theta^-}}{\mathrm dt}\right).
  $$  The unstable component is mainly the time derivative inside $\mathrm dF_{\theta^-}/\mathrm dt$, rather than the input Jacobian or the ODE velocity (surprisingly, as Jacobians tend to be nasty).
* They decompose the problematic time derivative as
$$
  \sin(t)\partial_tF_{\theta^-}
  =
  \sin(t)
  \frac{\partial c_{\mathrm{noise}}}{\partial t}
  \frac{\partial\operatorname{emb}}{\partial c_{\mathrm{noise}}}
  \frac{\partial F_{\theta^-}}{\partial\operatorname{emb}}.
  $$

  This decomposition motivates three architectural changes.

* **Use the identity time transformation.** Translating the EDM parameterization into TrigFlow gives  $$
  c_{\mathrm{noise}}(t)=\log\!\left(\sigma_d\tan(t)\right),
  \qquad
  \sin(t)c_{\mathrm{noise}}'(t)=\frac{1}{\cos(t)}.
  $$This diverges as $t\to\pi/2$. They instead use  $$
  c_{\mathrm{noise}}(t)=t.
  $$
* **Use positional timestep embeddings instead of high-scale Fourier embeddings.** For a generic sinusoidal feature,
$$
  \operatorname{emb}(c)
  =
  \sin\!\left(s\,2\pi\omega c+\phi\right),
  \qquad
  \partial_c\operatorname{emb}(c)
  =
  s\,2\pi\omega\cos\!\left(s\,2\pi\omega c+\phi\right).
  $$

  A large Fourier scale $s$ directly amplifies and makes the time derivative highly oscillatory. The positional embeddings used in the paper are approximately equivalent to using a small Fourier scale $s\approx0.02$, producing much smoother timestep derivatives.

* After the TrigFlow and architecture changes, the continuous-time CM gradient becomes  $$
  \nabla_\theta\mathbb E\left[
  -w(t)\sigma_d\sin(t)
  F_\theta^\top
  \frac{\mathrm df_{\theta^-}}{\mathrm dt}
  \right].
  $$  Most remaining gradient variance comes from the tangent itself.

* **Tangent normalization** explicitly limits this variance. In the implementation they work with
$$
  g=\cos(t)\frac{\mathrm df_{\theta^-}}{\mathrm dt},
  \qquad
  \bar g=\frac{g}{\|g\|+c},
  \qquad
  c=0.1.
  $$  Elementwise clipping to $[-1,1]$ also works, but normalization performs well without imposing a hard coordinate-wise threshold.

* **Adaptive weighting** replaces a manually designed $w(t)$ with a learned scalar function $w_\phi(t)$. The useful identity is
$$
  \nabla_\theta\mathbb E[F_\theta^\top y]
  =
  \frac12\nabla_\theta
  \mathbb E\left[\|F_\theta-F_{\theta^-}+y\|_2^2\right],
  $$
  where $y$ is independent of $\theta$. This converts the continuous consistency gradient into an MSE-like objective: $$
  \mathcal L_{\mathrm{sCM}}
  =
  \mathbb E\left[
  \frac{e^{w_\phi(t)}}{D}
  \left\|
  F_\theta-F_{\theta^-}-\bar g
  \right\|_2^2
  -w_\phi(t)
  \right].
  $$ Although $F_\theta$ and $F_{\theta^-}$ have the same forward value, gradients only pass through $F_\theta$. The learned weight attempts to equalize loss scales across noise levels. They also incorporate the prior weighting  $$
  w_{\mathrm{prior}}(t)=\frac{1}{\sigma_d\tan(t)}.
  $$ The same algorithm supports both training from data and distillation. The required trajectory velocity is

  $$
  \frac{\mathrm dx_t}{\mathrm dt}
  =
  \begin{cases}
  \cos(t)z-\sin(t)x_0, & \text{sCT},\\[2mm]
  \sigma_dF_{\mathrm{pretrain}}
  \left(\dfrac{x_t}{\sigma_d},t\right), & \text{sCD}.
  \end{cases}
  $$

  Thus, sCT uses an unbiased velocity estimator from $(x_0,z)$, while sCD follows the ODE of a pretrained diffusion teacher.

* When finetuning from a diffusion model, they use **tangent warmup**:

  $$
  r=\min\!\left(1,\frac{\text{iteration}}{H}\right),
  \qquad H=10{,}000,
  $$  $$
  g=
  -\cos^2(t)\left(\sigma_dF_{\theta^-}-\frac{\mathrm dx_t}{\mathrm dt}\right)
  -r\cos(t)\sin(t)
  \left(x_t+\sigma_d\frac{\mathrm dF_{\theta^-}}{\mathrm dt}\right).
  $$

  The initially unstable derivative term is therefore introduced gradually rather than at full strength from the first iteration.

* Computing $\mathrm dF/\mathrm dt$ requires a Jacobian-vector product. A direct JVP can overflow in FP16 near $t=0$ or $t=\pi/2$, so they rearrange the scaled derivative:  $$
  \cos(t)\sin(t)\frac{\mathrm dF}{\mathrm dt}
  =
  \nabla_{x_t/\sigma_d}F\cdot
  \left(\cos(t)\sin(t)\frac{\mathrm dx_t}{\mathrm dt}\right)
  +
  \partial_tF\left(\cos(t)\sin(t)\sigma_d\right).
  $$  The trigonometric factors are moved inside the JVP, preventing excessively large intermediate tangents. They additionally implement a Flash-Attention-style operation that computes attention and its JVP together in one memory-efficient pass.

* One-step sampling directly evaluates the consistency map:  $$
  \hat x_0=f_\theta(x_T,T),
  \qquad
  x_T\sim\mathcal N(0,\sigma_d^2I).
  $$  For two-step sampling, the first prediction is re-noised at an intermediate time and denoised again: $$
  x_{t_{\mathrm{mid}}}
  =
  \cos(t_{\mathrm{mid}})\hat x_0
  +\sin(t_{\mathrm{mid}})z,
  \qquad
  \hat x_0\leftarrow f_\theta(x_{t_{\mathrm{mid}}},t_{\mathrm{mid}}).
  $$
  They use $t_{\mathrm{mid}}=1.1$. In the experiments, sampling starts from
$$
  t_{\max}=\arctan\!\left(\frac{\sigma_{\max}}{\sigma_d}\right),
  \qquad
  \sigma_{\max}=80.
  $$
* In the discrete-time ablation, increasing the number of discretization points initially improves quality by reducing ODE error, but performance degrades beyond roughly $N=1024$ because the finite differences become numerically unreliable. Continuous-time training avoids this trade-off and outperforms the tested discrete variants.

* Seems to be more diverse
## I+D
-

## Deep Dive

