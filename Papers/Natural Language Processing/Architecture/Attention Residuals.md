---
state: skimmed
name: Attention Residuals
link: https://arxiv.org/abs/2603.15031v1
tldr: Did an attention mechanism over the residual stream, mixing hidden states from all layers more effectively
note:
quality:
  - good
abstract: |
  Residual connections with PreNorm are standard in modern LLMs, yet they accumulate all layer outputs with fixed unit weights. This uniform aggregation causes uncontrolled hidden-state growth with depth, progressively diluting each layer's contribution. We propose Attention Residuals (AttnRes), which replaces this fixed accumulation with softmax attention over preceding layer outputs, allowing each layer to selectively aggregate earlier representations with learned, input-dependent weights. To address the memory and communication overhead of attending over all preceding layer outputs for large-scale model training, we introduce Block AttnRes, which partitions layers into blocks and attends over block-level representations, reducing the memory footprint while preserving most of the gains of full AttnRes. Combined with cache-based pipeline communication and a two-phase computation strategy, Block AttnRes becomes a practical drop-in replacement for standard residual connections with minimal overhead.
    Scaling law experiments confirm that the improvement is consistent across model sizes, and ablations validate the benefit of content-dependent depth-wise selection. We further integrate AttnRes into the Kimi Linear architecture (48B total / 3B activated parameters) and pre-train on 1.4T tokens, where AttnRes mitigates PreNorm dilution, yielding more uniform output magnitudes and gradient distribution across depth, and improves downstream performance across all evaluated tasks.
paper id: 2603.15031v1
authors:
  - Kimi Team
  - Guangyu Chen
  - Yu Zhang
  - Jianlin Su
  - Weixin Xu
  - Siyuan Pan
  - Yaoyu Wang
  - Yucheng Wang
  - Guanduo Chen
  - Bohong Yin
  - Yutian Chen
  - Junjie Yan
  - Ming Wei
  - Y. Zhang
  - Fanqing Meng
  - Chao Hong
  - Xiaotong Xie
  - Shaowei Liu
  - Enzhe Lu
  - Yunpeng Tai
  - Yanru Chen
  - Xin Men
  - Haiqing Guo
  - Y. Charles
  - Haoyu Lu
  - Lin Sui
  - Jinguo Zhu
  - Zaida Zhou
  - Weiran He
  - Weixiao Huang
  - Xinran Xu
  - Yuzhi Wang
  - Guokun Lai
  - Yulun Du
  - Yuxin Wu
  - Zhilin Yang
  - Xinyu Zhou
publication date: 2026-03-16T09:32
comments: attnres tech report
pdf: https://arxiv.org/pdf/2603.15031v1
tags:
  - llm
  - architecture
---
#paper
## Takeaways
-

## I+D
-

## Deep Dive

