---
state: read
name: "Safer by Diffusion, Broken by Context: Diffusion LLM's Safety Blessing and Its Failure Mode"
link: https://arxiv.org/abs/2602.00388v2
tldr: Analysed how DLMs are more robust to simple black-box attacks due to their denoising mechanics (based on mild assumptions). Also showed that DLMs are fragile to context-infilling (where the harmful response is mixed with some formatting task, e.g. json, Python, MD)
note:
quality:
  - good
abstract: "Diffusion large language models (D-LLMs) offer an alternative to autoregressive LLMs (AR-LLMs) and have demonstrated advantages in generation efficiency. Beyond the utility benefits, we argue that D-LLMs exhibit a previously underexplored safety blessing: their diffusion-style generation confers intrinsic robustness against jailbreak attacks originally designed for AR-LLMs. In this work, we provide an initial analysis of the underlying mechanism, showing that the diffusion trajectory induces a stepwise reduction effect that progressively suppresses unsafe generations. This robustness, however, is not absolute. Following this analysis, we highlight a simple yet effective failure mode, context nesting, in which harmful requests are embedded within structured benign contexts. Empirically, we show that this simple black-box strategy bypasses D-LLMs' safety blessing, achieving state-of-the-art attack success rates across models and benchmarks. Notably, it enables the first successful jailbreak of Gemini Diffusion to our knowledge, exposing a critical vulnerability in proprietary D-LLMs. Together, our results characterize both the origins and the limits of D-LLMs' safety blessing, constituting an early-stage red-teaming of D-LLMs."
paper id: 2602.00388v2
authors:
  - Zeyuan He
  - Yupeng Chen
  - Lang Lin
  - Yihan Wang
  - Shenxu Chang
  - Eric Sommerlade
  - Philip Torr
  - Junchi Yu
  - Adel Bibi
  - Jialin Yu
publication date: 2026-01-30T23:08
comments: ""
pdf: https://arxiv.org/pdf/2602.00388v2
tags:
  - safety
  - dlms
---
#paper
## Takeaways
- DLMs are more robust to black-box jailbreaking, this is justified by their sequential denoising nature
- To sustain the previous claims the authors define the distance from a safe vocabulary $\mathcal{S}$ as: $D(x_t, \mathcal{S}) := 1- \mathbb{E}_{x_{0} \sim p(x_{0}|x_{t})}[\mathbb{1}_{x_0 \in \mathcal{S}}|x_{t}]$, I abused of notation, but I basically mean the expected value of a clean response being completely covered by $\mathcal{S}$ starting from $x_t$. Intuitively their metric is really expensive, as it requires many rollouts of denoising to even perform a Monte Carlo estimation
- Assuming that at each denoising step the distance stays the same or decreases (i.e. the overall response gets safer), they proved that one could bound the overall distance of the response with a selected $\epsilon$, thus determining how many denoising steps are needed to get to that safety level overall. While the proof is right, it also depends on another parameter $\delta$ which is an assumed upper bound, which in practice we don't know. So the proof is rather decorative than useful. And it's tightly related to the safe vocabulary choice
- Finally, they conducted a weak experiment to show that:
	- DLMs reduce the distance from the safe vocabulary as denoising steps progress, "validating" their assumption
	- Showed DLMs are more robust than GPT-o4 for some black-box attacks (maybe GPT-4 was completely cherry-picked)
	- Showed DLMs are relatively easy to jailbreak with some context-prefilling in which the model gets a mixed prompt asking for harmful stuff but also format things in a certain way (json, markdown, code, etc)
- Strongest finding is the robustness to black-box attacks and easiness of jailbreaks with the context-prefilling

## I+D
-

## Deep Dive

