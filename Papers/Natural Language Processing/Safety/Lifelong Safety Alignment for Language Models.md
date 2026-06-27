---
state: skimmed
name: Lifelong Safety Alignment for Language Models
link: https://arxiv.org/abs/2505.20259v1
tldr: Proposed adversarial play of attacker and defender
note:
quality:
  - good
abstract: "LLMs have made impressive progress, but their growing capabilities also expose them to highly flexible jailbreaking attacks designed to bypass safety alignment. While many existing defenses focus on known types of attacks, it is more critical to prepare LLMs for unseen attacks that may arise during deployment. To address this, we propose a lifelong safety alignment framework that enables LLMs to continuously adapt to new and evolving jailbreaking strategies. Our framework introduces a competitive setup between two components: a Meta-Attacker, trained to actively discover novel jailbreaking strategies, and a Defender, trained to resist them. To effectively warm up the Meta-Attacker, we first leverage the GPT-4o API to extract key insights from a large collection of jailbreak-related research papers. Through iterative training, the first iteration Meta-Attacker achieves a 73% attack success rate (ASR) on RR and a 57% transfer ASR on LAT using only single-turn attacks. Meanwhile, the Defender progressively improves its robustness and ultimately reduces the Meta-Attacker's success rate to just 7%, enabling safer and more reliable deployment of LLMs in open-ended environments. The code is available at https://github.com/sail-sg/LifelongSafetyAlignment."
paper id: 2505.20259v1
authors:
  - Haoyu Wang
  - Zeyu Qin
  - Yifei Zhao
  - Chao Du
  - Min Lin
  - Xueqian Wang
  - Tianyu Pang
publication date: 2025-05-26T17:40
comments: ""
pdf: https://arxiv.org/pdf/2505.20259v1
tags:
  - safety
  - llm
  - cl
---
#paper
## Takeaways
- Scraped jailbreaking techniques from papers using closed-source LLM (ChatGPT; interesting that GPT complied to give the techniques if framed as research-oriented work). With those base techniques an attacker model crafts tuples (strategy, prompt, goal) and inputs the prompts to the defender model. Upon rejection (classified by LLama-guard and Qwen as judge also) the feedback is returned to the attacker to update the attack techniques. If the attacker succeeds, the response increases a counter, and once the counter gets above a certain number

## I+D
-

## Deep Dive

