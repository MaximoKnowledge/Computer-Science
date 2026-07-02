## Core idea

This is a separate project that may build on Project 1. Instead of sending a compact message directly to the receiver, the sender produces an opaque or compressed code $z$, which is expanded at the receiver endpoint into a richer context or instruction representation.

$$
s_A \rightarrow E_\theta \rightarrow z
$$

$$
z \rightarrow D_{\phi, B} \rightarrow c_B
$$

$$
\hat{y} = B(q, c_B)
$$

Here, $E_\theta$ compresses the sender state into a compact code, while $D_{\phi, B}$ expands that code into a form useful for receiver $B$.

## Relation to Project 1

Project 1 studies direct reward-optimized message compression:

$$
A \rightarrow E_\theta \rightarrow B
$$

Project 2 adds an endpoint expansion mechanism:

$$
A \rightarrow E_\theta \rightarrow z \rightarrow D_{\phi, B} \rightarrow B
$$

Thus, Project 2 can be viewed as a receiver-side extension of Project 1.

## Objective

The system is trained so that the compressed code and endpoint expander jointly preserve the task-relevant information needed by the receiver:

$$
\max_{\theta,\phi} \; \mathbb{E}\left[
R(B(q, D_{\phi,B}(E_\theta(s_A))), y^\*) - \lambda |z|
\right]
$$

where $z = E_\theta(s_A)$ is the compact transmitted code.

## Interpretation

The transmitted code $z$ may be opaque to humans and may function as a private learned protocol between endpoints. However, unless explicit adversarial secrecy guarantees are defined and evaluated, it should not be called cryptographic encryption.

More precise terms include:

- learned semantic code;
- opaque inter-agent protocol;
- compressed endpoint-expanded message;
- receiver-conditioned code.

## Main contribution

This project studies whether a compact learned code can be expanded at the receiver endpoint into a richer context that enables downstream task success. Compared with Project 1, it separates transmission efficiency from receiver usability by introducing a learned expansion layer.

## Minimal experimental setup

Use a fixed sender state $s_A$, a learned encoder $E_\theta$, a learned receiver-specific expander $D_{\phi,B}$, and a frozen receiver model $B$.

Evaluate against:

- direct compressed message from Project 1;
- natural-language summaries;
- autoencoder-style reconstruction;
- fixed symbolic codes;
- latent or soft-prompt compression baselines;
- semantic communication baselines.

## Claim

Endpoint expansion enables extremely compact inter-agent communication by allowing the transmitted message to be optimized as a learned semantic code rather than as a directly interpretable message.

## Related work

This project is related to semantic communication, learned context compression, latent communication between language models, and endpoint-side representation reconstruction.

Semantic communication provides the broader communication-theoretic analogue: the goal is to transmit task-relevant meaning rather than preserve exact symbols or bits. DeepSC is a representative deep-learning-based semantic communication system for text transmission. Gist Tokens and AutoCompressors are also relevant because they compress context into learned representations. Recent latent-communication work studies direct exchange of continuous representations, hidden states, or KV-cache information between language-model agents instead of natural-language messages.

## References

- Qin, Z., Tao, X., Lu, J., Tong, W., & Li, G. Y. (2021). *Semantic Communications: Principles and Challenges*. arXiv:2201.01389. https://arxiv.org/abs/2201.01389
- Xie, H., Qin, Z., Li, G. Y., & Juang, B.-H. (2020). *Deep Learning Enabled Semantic Communication Systems*. arXiv:2006.10685. https://arxiv.org/abs/2006.10685
- Mu, J., Li, X. L., & Goodman, N. (2023). *Learning to Compress Prompts with Gist Tokens*. arXiv:2304.08467. https://arxiv.org/abs/2304.08467
- Chevalier, A., Wettig, A., Ajith, A., & Chen, D. (2023). *Adapting Language Models to Compress Contexts*. arXiv:2305.14788. https://arxiv.org/abs/2305.14788
- Wu, S. et al. *Learning Communication between Language Models*. OpenReview. https://openreview.net/forum?id=rJcMv7q1jH
- Zheng, Y. et al. *Thought Communication in Multiagent Collaboration*. OpenReview. https://openreview.net/forum?id=tq9lyV9Cml
- Du, Z. et al. *Enabling Agents to Communicate Entirely in Latent Space*. OpenReview. https://openreview.net/forum?id=rmYbgsehTd
- Kriuk, B., & Ng, L. (2025). *Q-KVComm: Efficient Multi-Agent Communication via Adaptive KV Cache Compression*. arXiv:2512.17914. https://arxiv.org/abs/2512.17914