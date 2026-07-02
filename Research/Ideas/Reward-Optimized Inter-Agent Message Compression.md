## Core idea

We study a setting where one language-model agent $A$ calls another language-model agent $B$ through an intermediate learned encoder $E_\theta$. The encoder receives the sender agent's state, intent, or intermediate work, and produces a compact message that is passed directly to the receiving agent.

$$
s_A = A(x)
$$

$$
m = E_\theta(s_A, q, B)
$$

$$
\hat{y} = B(q, m)
$$

The goal is not to reconstruct $s_A$, nor to produce a human-readable summary. The goal is to optimize the message $m$ so that the downstream receiver $B$ performs the target task correctly under a communication budget.

## Objective

The encoder is trained using downstream reward from the receiver:

$$
\max_\theta \; \mathbb{E}\left[
R(B(q, E_\theta(s_A, q, B)), y^\*) - \lambda |E_\theta(s_A, q, B)|
\right]
$$

where $R$ measures task success and $|m|$ penalizes message length.

## Method

The sender $A$ and receiver $B$ can remain frozen. Only the intermediate encoder $E_\theta$ is optimized. A practical training method is reward-guided optimization, such as GRPO or another policy-gradient-style objective. Multiple candidate messages are sampled from $E_\theta$, evaluated by running the downstream receiver $B$, and updated according to task reward.

## Main contribution

This project optimizes inter-agent communication directly for downstream utility. Unlike prompt compression, the compressed object is not merely a prompt or context. It is the communication payload generated when one agent calls another.

## Minimal experimental setup

Use a two-agent system:

$$
A \rightarrow E_\theta \rightarrow B
$$

The sender produces a verbose intermediate state $s_A$. The encoder compresses this state into a short message $m$. The receiver uses $m$ to solve the task.

Baselines include:

- full sender message;
- no sender message;
- natural-language summary;
- structured JSON;
- generic prompt compression;
- supervised compression;
- fixed hand-designed protocol;
- discrete prompt optimization.

The main evaluation is task performance as a function of message length.

## Claim

A learned encoder trained with downstream reward can discover compact, receiver-effective inter-agent messages that outperform generic summarization or hand-designed communication formats under tight token budgets.

## Related work

This project is related to prompt compression, learned prompt representations, discrete prompt optimization, and learned communication in multi-agent systems.

Prompt compression methods such as LLMLingua compress long prompts to reduce inference cost while preserving downstream performance. Gist Tokens and AutoCompressors study learned prompt or context compression using special tokens or soft summary vectors. RLPrompt is relevant because it optimizes discrete prompts through reinforcement learning, showing that effective prompts need not be natural or human-readable. GRPO provides a practical reward-optimization template for updating the encoder from downstream task rewards. Earlier multi-agent RL work such as RIAL/DIAL and CommNet shows that communication protocols can be learned end-to-end under task reward.

## References

- Jiang, H., Wu, Q., Lin, C.-Y., Yang, Y., & Qiu, L. (2023). *LLMLingua: Compressing Prompts for Accelerated Inference of Large Language Models*. arXiv:2310.05736. https://arxiv.org/abs/2310.05736
- Mu, J., Li, X. L., & Goodman, N. (2023). *Learning to Compress Prompts with Gist Tokens*. arXiv:2304.08467. https://arxiv.org/abs/2304.08467
- Chevalier, A., Wettig, A., Ajith, A., & Chen, D. (2023). *Adapting Language Models to Compress Contexts*. arXiv:2305.14788. https://arxiv.org/abs/2305.14788
- Deng, M., Wang, J., Hsieh, C.-P., Wang, Y., Guo, H., Shu, T., Song, M., Xing, E. P., & Hu, Z. (2022). *RLPrompt: Optimizing Discrete Text Prompts with Reinforcement Learning*. EMNLP 2022. https://aclanthology.org/2022.emnlp-main.222/
- Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Bi, X., Zhang, H., Zhang, M., Li, Y. K., Wu, Y., & Guo, D. (2024). *DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models*. arXiv:2402.03300. https://arxiv.org/abs/2402.03300
- Foerster, J. N., Assael, Y. M., de Freitas, N., & Whiteson, S. (2016). *Learning to Communicate with Deep Multi-Agent Reinforcement Learning*. arXiv:1605.06676. https://arxiv.org/abs/1605.06676
- Sukhbaatar, S., Szlam, A., & Fergus, R. (2016). *Learning Multiagent Communication with Backpropagation*. arXiv:1605.07736. https://arxiv.org/abs/1605.07736
- Tran, K.-T., Dao, D., Nguyen, M.-D., Pham, Q.-V., O'Sullivan, B., & Nguyen, H. D. (2025). *Multi-Agent Collaboration Mechanisms: A Survey of LLMs*. arXiv:2501.06322. https://arxiv.org/abs/2501.06322