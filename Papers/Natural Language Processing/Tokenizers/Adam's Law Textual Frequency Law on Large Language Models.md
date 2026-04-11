---
state: skimmed
name: "Adam's Law: Textual Frequency Law on Large Language Models"
link: https://arxiv.org/abs/2604.02176v2
tldr: Low-frequency text degrades performance (bad paper, good idea)
note:
quality:
  - meh
abstract: While textual frequency has been validated as relevant to human cognition in reading speed, its relatedness to Large Language Models (LLMs) is seldom studied. We propose a novel research direction in terms of textual data frequency, which is an understudied topic, to the best of our knowledge. Our framework is composed of three units. First, this paper proposes Textual Frequency Law (TFL), which indicates that frequent textual data should be preferred for LLMs for both prompting and fine-tuning. Since many LLMs are closed-source in their training data, we propose using online resources to estimate the sentence-level frequency. We then utilize an input paraphraser to paraphrase the input into a more frequent textual expression. Next, we propose Textual Frequency Distillation (TFD) by querying LLMs to conduct story completion by further extending the sentences in the datasets, and the resulting corpora are used to adjust the initial estimation. Finally, we propose Curriculum Textual Frequency Training (CTFT) that fine-tunes LLMs in an increasing order of sentence-level frequency. Experiments are conducted on our curated dataset Textual Frequency Paired Dataset (TFPD) on math reasoning, machine translation, commonsense reasoning and agentic tool calling. Results show the effectiveness of our framework.
paper id: 2604.02176v2
authors:
  - Hongyuan Adam Lu
  - Z. L.
  - Victor Wei
  - Zefan Zhang
  - Zhao Hong
  - Qiqi Xiang
  - Bowen Cao
  - Wai Lam
publication date: 2026-04-02T15:39
comments: ACL 2026 Main Conference
pdf: https://arxiv.org/pdf/2604.02176v2
tags:
  - llm
---
#paper
## Takeaways
-

## I+D
- Can we create a prompt translator? From a low-frequency one to a high-energy one? This would basically get free performance (reducing stochasticity of prompts)

## Deep Dive

