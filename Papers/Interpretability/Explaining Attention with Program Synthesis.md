---
state: to-read
name: Explaining Attention with Program Synthesis
link: https://arxiv.org/abs/2606.19317v2
tldr: They make Python programs that replicate the behaviour of attention maps
note:
quality:
  - banger
abstract: "A longstanding goal of research on interpretable deep learning is to replace opaque neural computations with human-meaningful symbolic descriptions. In this paper, we propose an approach for approximating the behavior of components of deep networks with executable programs. We focus on attention heads in transformer language models. For a given head, we first compute its associated attention matrices on a collection of randomly selected training examples. Next, we prompt a pre-trained language model with a summary of these matrices, and instruct it to generate a set of Python programs that can reproduce the associated attention patterns given only text from the input sentence. Finally, we re-rank programs according to how well our final set of programs predict behavior on held-out inputs. We demonstrate that a set of fewer than 1,000 such generated programs can reproduce the attention patterns of heads in GPT-2, TinyLlama-1.1B, and Llama-3B, achieving an average Intersection-over-Union similarity above 75% on TinyStories. Moreover, the best-fit programs can replace neural attention heads without substantially affecting model behavior: replacing 25% of attention heads with programmatic surrogates across the three models incurs only a 16% average perplexity increase, while maintaining performance on a variety of downstream question answering benchmarks. This work contributes a scalable pipeline for reverse-engineering attention heads in transformer models using human-readable, executable code, advancing a path toward symbolic transparency in neural models."
paper id: 2606.19317v2
authors:
  - Amiri Hayes
  - Belinda Z Li
  - Jacob Andreas
publication date: 2026-06-17T17:40
comments: ""
pdf: https://arxiv.org/pdf/2606.19317v2
tags:
  - xai
  - llm
  - nlp
---
#paper
## Takeaways
- Extremely simple and good method: take attention maps and pair them with the input sequence. Then make an LLM-loop where you ask the model to write a Python program that based on the input tokens reproduces the same attention map (i.e. matching the specific weights each token assigns to each other). They measure the JSD to see which programs predict closer maps to others and iterate with the model until they believe the program is faithful enough. 
- To evaluate the synthesised program they do two tests:
	- Measure the IoU (in a strange way) with the original attention map
	- Replace the head with the program and measure downstream performance
	The evaluation is pretty elegant and done on held-out attention maps.
- Overall their method works pretty good and they demonstrate that you can synthesise quite a bunch of heads for small models. Encoder models remain harder to synthesise due to bidirectional attention

## I+D
- The baseline seems not so robust. What does deactivating the head at all produce in the model? The head's utility gets confounded with the method's precision. Putting a head OOD seems not the best baseline to compare against

- There are some outliers of heads that despite being replaced still change the perplexity substantially. What's the overall causal importance of those heads?

They acknowledge this functionality gap in the limitations. Which puts into great risk the whole approach's out-of-the-box performance. They literally say that the highest IoU come from extremely simple programs and they may be confounding the head's functional relevance with their method's performance.

## Deep Dive

