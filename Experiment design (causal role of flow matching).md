Date: 04/08/2026
Status: Draft for discussion
Scope: assessment of the current project state, then a design for the causal claim that flow matching contributes to consistency modelling, then the generalisation axes needed to state that claim safely.

Two open decisions are listed at the end of Part 1. Part 2 revises the "cut CIFAR/CelebA" line in Part 1's cut list.

---

# Part 1 — State of the project and the causal design

## Where the project actually stands

Three findings, and what each is worth:

| Finding | Real? | Load-bearing? |
| --- | --- | --- |
| LSD explodes below η=0.375; PSD never does | **Yes** — 5 seeds, reproducible, survives 100k+ steps then collapses in one step | Yes, but as a *stability* result, not a representation one |
| PSD's η-optimum is interior (0.375, then 0.25) | **Probably** | This is the whole "FM helps" claim, and it is currently unidentified (see below) |
| LSD wins 1-step KL, PSD wins 16-step | **Half.** In `image 6.png` the 16-step bands are cleanly separated. In `image 5.png` the 1-step bands overlap for the entire run | Drop the 1-step half until it survives error bars |

The infrastructure is in better shape than the docs suggest: `val_eval_{lsd,psd_uniform,esd}_special_raw` is already logged for every run, so cross-family residuals — the expensive part of every design below — already exist.

## Three things to fix before designing anything new

**1. The KL estimator floor.** The 1-step LSD sweep spans 0.095–0.115 across all seven η. The 16-step curves bottom out near 0.08–0.09. Compute self-KL — the estimator applied to 25k *true* checker samples against 25k more — and that is the floor. If it is ~0.09, then every intra-family 1-step ranking currently written down is noise, and §3 and §4 of the η-ablation summary need rewriting. One hour of work, and it decides how much of the existing log survives.

**2. The cosine-similarity evidence is measuring initialisation, not loss.** In `Preliminary tests of geometry/image 4.png`, three of four boxes sit at ~0.0: `lsd: different seeds`, `psd: different seeds`, `cross-group: different seeds`. Only `cross-group: same seed` is elevated (~0.5). Raw per-neuron cosine between independently initialised networks is ~0 *by construction* — there is no neuron correspondence. So "different seeds" is a floor, not a baseline, and the only thing the plot shows is that shared init keeps two models in the same basis. It says nothing about whether the losses induce different structure.

Two consequences:

- The Evidence row `Same seed different loss has higher cosine similarity…` is marked **Supports**, but its own Interpretation field ("different losses are actually inducing pretty similar gradient updates") *contradicts* the claim it is attached to. Flip it to Contradicts or Mixed.
- The claim cannot be revisited without a permutation-invariant measure (CKA / Procrustes / SVCCA), which the `Lacks` field already names.

**3. η is a confounded treatment, and this is the blocker.** Lowering η simultaneously (a) reduces diagonal samples → degrades $v_{t,t}$, (b) increases consistency samples, (c) changes gradient variance, (d) approaches the regime where the identity map satisfies PSD exactly. One knob, four channels. No value of the η curve identifies "flow matching helps", because the interior optimum is fully explained by (d) alone — a boundary anchor you need *some* of and nothing more.

> This is why the causal proof keeps slipping away: the intervention is on the diagonal's *share*, and the claim is about the diagonal's *content*.

## What the claim has to be decomposed into

| | Statement | Status |
| --- | --- | --- |
| **C0** | Without $\mathcal L_b$, PSD collapses to identity ($X_{s,t}=x$ satisfies semigroup exactly) | Trivially true. Run it once as a control; it is not a result |
| **C1** | Map quality is monotone in *diagonal quality*, at fixed consistency budget | ← **the actual claim.** Untested |
| **C2** | The channel is information transfer, not variance reduction or stabilisation | Untested |
| **C3** | The transfer rate differs between LSD and PSD | This is the paper's differential |

And the observation that should be the spine of the paper:

> $\mathcal L_{\mathrm{PSD}}$ is a pure function of $\hat X$ and **never evaluates $\hat v_{t,t}$**. $\mathcal L_{\mathrm{LSD}}$ evaluates it explicitly, inside its own residual.

So for LSD, "flow matching helps" is near-tautological — its target *is* the model's diagonal. For PSD there is no term-level coupling at all: the only routes from $\mathcal L_b$ to the off-diagonal map are (i) breaking the identity degeneracy and (ii) shared parameters. **PSD is therefore the clean testbed, and the answer is genuinely not obvious.** That framing is sharper than "different losses, different geometries", and it makes C3 a differential rather than two separate stories.

## Build this first: the oracle velocity

The checker is a union of axis-aligned squares and the base is Gaussian, so with $I_t=(1-t)x_0+tx_1$:

$$
p_t(x) \propto \int_S N(x;\,tu,\,(1-t)^2\sigma^2 I)\,du, \qquad b_t(x)=\frac{\mathbb E[x_1\mid I_t=x]-x}{1-t}
$$

Both integrals factorise per coordinate over each rectangle → closed form in $\Phi$ and $\phi$ (truncated-Gaussian moments). Exact, cheap, vectorised. Integrate $\dot x = b_t(x)$ with a tight-tolerance adaptive solver and the **true flow map** $X^\star_{s,t}$ follows.

This is an afternoon of work and it upgrades every measurement in the project:

- $\mathcal E(s,t)=\mathbb E_{x\sim p_s}\lVert \hat X_{s,t}(x)-X^\star_{s,t}(x)\rVert^2$ — map error against ground truth, resolved by $|t-s|$, instead of sample-quality proxies with an unmeasured floor
- $\mathcal E_{\mathrm{diag}}(t)=\mathbb E\lVert\hat v_{t,t}-b_t\rVert^2$ — an absolute measure of flow-matching quality, comparable across arms
- Together they let C1 be stated as a **coefficient**: *a unit reduction in diagonal velocity error yields β units of flow-map error reduction at $|t-s|=1$, at fixed consistency budget.*

Nothing else should run until this exists. Everything below assumes it.

## The experiments

### E1 — δ-tracer: inject a known signal into the diagonal, measure what arrives off-diagonal

**This is the causal proof.** Fix η=0.5 and the batch split exactly. The only thing that varies across arms is *what the diagonal regresses toward*:

| Arm | Diagonal target | Role |
| --- | --- | --- |
| A0 | $\dot I_t$ | standard reference |
| A1 | $b^\star(I_t)$ | oracle mean, low variance |
| A2 | $b^\star(I_t)+\epsilon$, $\mathrm{Cov}(\epsilon)$ matched to $\mathrm{Cov}(\dot I_t\mid I_t)$ | **the honest control for A0** — same mean, same variance, known ground truth |
| A3 | $b^\star + \lambda\delta$, $\delta$ smooth, localised in an $(x,t)$ band, $\lambda\in\{0.1,0.3,1.0\}$ | the tracer |
| A4 | frozen $\eta{=}1$ checkpoints at 5k / 25k / 250k steps, + matched noise | teacher-quality ladder |

A2 matters: swapping $\dot I_t$ for $b^\star$ cuts the target variance enormously, so A1-vs-A0 confounds information with variance reduction. A2 removes that.

**What gets measured.** For A3, the *counterfactual* map $X^{\delta}$ — integrate $b^\star+\lambda\delta$ with the same solver. Then

$$
\gamma(s,t;\lambda) = \frac{\langle \hat X_{s,t}-X^\star_{s,t},\; X^{\delta}_{s,t}-X^\star_{s,t}\rangle}{\lVert X^{\delta}_{s,t}-X^\star_{s,t}\rVert^2}
$$

is the fraction of the injected perturbation that arrived. $\gamma\to1$ means the map fully inherited the corrupted flow; $\gamma\to0$ means the diagonal is a pure anchor.

At infinite capacity and convergence, $\gamma=1$ is forced — semigroup plus a diagonal velocity $\tilde b$ uniquely determines the flow of $\tilde b$. **That is the point.** The measurement is not whether it arrives, it is the *transport profile*: $\gamma$ as a function of $|t-s|$, of $\lambda$, and of training step. How much of the diagonal signal reaches $|t-s|=1$, and how many steps it takes. LSD should transport fast and locally; PSD slower and more diffusely, because it only has parameter sharing to carry it. **That difference is C3, measured directly.**

Placebo arm: inject $\delta$ into the *consistency* term instead and measure reverse transport. The result is a 2×2 transport matrix, a much stronger object than "PSD seems orthogonal, LSD seems to interfere".

**Kills the claim:** $\gamma$ flat near 0 at $|t-s|>0.5$ for PSD across all $\lambda$ and all of training → flow matching is an anchor for PSD and nothing more. That is a publishable negative and it ends the thread cleanly.

### E2 — deconfound the η sweep: $N_{\mathrm{diag}} \times N_{\mathrm{cons}}$ grid

Stop coupling the two sample counts. Grid $N_{\mathrm{diag}}, N_{\mathrm{cons}} \in \{12.5\text{k}, 25\text{k}, 50\text{k}\}$ as independent axes (batch size varies; on 2-D checker that is affordable). The current sweep is the anti-diagonal of this grid.

- Anchor-only → quality depends on $N_{\mathrm{cons}}$, flat in $N_{\mathrm{diag}}$ above a small threshold
- Teacher → quality improves with $N_{\mathrm{diag}}$ at fixed $N_{\mathrm{cons}}$
- The **iso-quality contour slope is the exchange rate** between a flow-matching sample and a consistency sample — i.e. the H6 prescription, in FLOP units, falling out of the science runs. Denominate in FLOPs, not samples: a PSD off-diagonal sample costs two extra forwards, an LSD one costs a JVP.

Trim to an L-shape (5 cells) if compute is tight; the corner cells carry the argument.

### E3 — gradient routing: is the channel representational?

For PSD specifically. Restrict $\nabla\mathcal L_b$ to the top $k$ of 4 hidden layers, $k\in\{1,2,3,4\}$; $\mathcal L_D$ always trains everything. The diagonal still anchors the *function* $v_{t,t}$ at every $k$, but at small $k$ it cannot shape the trunk representation.

Flat in $k$ → FM's contribution is functional anchoring only. Monotone degradation → FM shapes the shared representation the consistency term reads from. This is the one experiment that connects the causal question back to the representation angle the project started from, and it is ~15 runs.

### Always-on logging (fold into all of the above)

- $\cos(\nabla\mathcal L_b,\nabla\mathcal L_D)$ per step, per family, per η. One extra backward at log cadence. This distinguishes **redundancy** (parallel) from **orthogonality** from **conflict** (negative) — and the AlphaFlow result cited in the proposal is a *conflict* claim, which is a different phenomenon from what the current log describes. Cheapest decisive plot in the project.
- Consistency residuals resolved by $|t-s|$ rather than integrated. Near the diagonal everything looks satisfied and the comparison is vacuous.
- Checkpoints + fixed-probe-batch activations at a regular cadence. Deciding later costs a rerun; logging costs disk.

### LSD stability — scope it to one subsection

A real differential finding, worth keeping, but it must not eat the project. Two diagnostics: (a) grad-norm and $\lVert\nabla_x\hat X\rVert$ traces in the 200 steps around a collapse, (b) LSD loss binned by $|t-s|$ to see whether large jumps drive it. The instantaneous onset after 100k healthy steps plus the failed 5k warmup both point at the self-referential structure $\hat v_{t,t}(\hat X_{s,t}(\cdot))$ — the model's output feeding its own target — not at insufficient flow-matching quality. Also: the 5k warmup at η=0.125 is a confound in the sweep; one no-warmup arm cleans it up.

## What to cut

Say it out loud so it stops reappearing: **split-time probing (H4), the FMM invertibility residual, the ESD appendix, PCA/t-SNE, CIFAR/CelebA.** The FMM residual would require training over $t<s$, changing the training distribution of every run in the paper, to measure something the NFE sweep measures better.

> *Revised in Part 2:* the CIFAR/CelebA line is wrong as stated. Images are not cut, they are sequenced — and the reason they can be sequenced is that the tracer metric turns out to be dataset-agnostic. See "What actually transfers to CIFAR".

## Order and budget

| | Experiment | Runs | Gate |
| --- | --- | --- | --- |
| 0 | Oracle + true-map metrics + KL floor check | 0 | Blocks everything |
| 1 | **E1 δ-tracer** | ~40 | If $\gamma$ flat for PSD → write the negative, stop |
| 2 | E2 budget grid | ~30 (L-shape) | Retro-fits the interpretation of the existing sweep |
| 3 | E3 gradient routing | ~15 | Only if E1 is positive |

~85 runs, against the ~70 already done. If only one thing runs, run E1.

## Two open decisions

1. Is the **δ-tracer** framing the one to use as the paper's causal spine, or the plainer teacher-quality ladder — less novel, easier to explain?
2. Does the LSD explosion become a subsection, or get deferred entirely?

---

# Part 2 — Which axes to ablate, and when to stop

Cutting CIFAR outright in Part 1 was wrong; the external-validity job it was doing is real, and the bootstrapped proposal already cites the precedent (Align Your Flow: the Lagrangian variant looked *more* stable on toys and failed on images). That is documented evidence of a sign flip in exactly this family. The question is not whether to scale, but what the axis list has to be and when to stop.

## The reframe that makes this finite

This is not sampling a population of datasets and architectures, so there is no $n$ at which generalisation becomes valid. It is testing a mechanism. Which means the stopping rule is not a count:

> **Stop when the list of alternative explanations a reviewer would name is empty.**

That list is finite and can be written today. Every axis earns its place by killing one entry. Anything that does not map to an entry is reassurance, and reassurance is what makes plans endless.

Second rule, which does most of the pruning: **only run an axis you can sign in advance.** If the direction can be stated — "the effect should get *larger* here, *smaller* there" — then a null is informative and a confirmation is a mechanism test. If it cannot, the run is not testing anything.

## The adversarial list

| # | "Your result is an artifact of…" | Plausible? | Killed by |
| --- | --- | --- | --- |
| 1 | …the time-embedding's smoothness. A good $v_{t,t}$ plus interpolation in $(s,t)$ already gives decent $v_{s,t}$ near the diagonal, with no consistency condition involved | **Very** — this could produce the entire effect | Time-conditioning axis |
| 2 | …$d=2$. In high dimensions the map is nowhere near converged and this reverses | **Very** — AYF precedent | Dimensionality axis |
| 3 | …infinite fresh data. With finite data the consistency term is a regulariser, not a teacher | **Yes** — currently `data_mode=fresh`, 25B samples, zero generalisation gap; images are the opposite regime | Data-regime axis |
| 4 | …the checkerboard. Disconnected support, extreme curvature | Moderate | Target-geometry axis |
| 5 | …capacity. A 4×512 MLP can satisfy both terms independently; a tight one could not | Moderate, and *signable* | Capacity axis |
| 6 | …the KL estimator floor | **Yes, currently** | Floor check |
| 7 | …the convex-stopgrad teacher | Moderate | Teacher-scheme arm (mechanism; folds into E1) |
| 8 | …survivorship. At η=0.375 only 3/5 LSD runs survived, so every LSD number there is conditioned on not exploding | **Yes, and already in the data** | Report conditioning explicitly; matched-survivor comparison |

#8 is not a future risk, it is a live defect in the η-ablation table. Any LSD-vs-PSD comparison at 0.25–0.375 compares all PSD seeds against surviving LSD seeds. Either report $n$ per cell and the survival rate as a result in its own right, or restrict cross-family comparisons to η ≥ 0.5 where both families are complete.

## The axes, with directions

| Axis | Levels | Predicted direction | Tier |
| --- | --- | --- | --- |
| **Dimensionality** | $d \in \{2, 16, 64\}$ | Farther from convergence ⇒ FM's *teaching* share rises relative to anchoring; $\gamma$ at large $\lvert t-s\rvert$ increases | 1 |
| **Time conditioning** | smooth embedding vs. coarse/binned or separate time-pair heads | If the effect is architectural smoothness, it collapses when interpolation is broken. If it is the loss, it survives | 1 |
| **Data regime** | fresh vs. capped dataset (e.g. $10^5$ samples) | Generalisation gap appears; consistency may switch from teacher to regulariser — the cheap proxy for "what happens on images" | 1 |
| **Target geometry** | checker / GMM well-separated / near-affine | Effect → 0 on the low-curvature target. **A positive control on the measurement** — if $\gamma$ is nonzero where the flow map is nearly affine, the metric is broken | 2 |
| **Coupling** | independent vs. minibatch-OT | Straighter paths ⇒ LSD's condition closer to free ⇒ redundancy asymmetry *widens*; PSD roughly unchanged | 2 |
| **Capacity** | 4×512 vs. 2×128 | Forced parameter sharing ⇒ transfer coefficient *increases* | 2 |
| **Real images** | CIFAR-10, DiT/EDM2 | Pattern preserved in sign, not magnitude | 3 |

Every row has a direction. That is the filter — anything that column cannot be filled in for does not get run.

## Three design choices that cut the budget hard

**Star, not grid.** Pick a centre cell (checker, $d{=}2$, fresh, MLP 4×512, independent) and move one axis at a time. That is $1 + \sum_j (L_j-1)$ cells ≈ 9, not $\prod_j L_j$ ≈ 200. Escalate to a two-factor interaction only when a specific axis actually moves the result — unearned interactions are where these budgets die.

**Pair the seeds.** The current design is unpaired, which is why 5 seeds still could not resolve a 0.01 KL gap in `image 5.png`. The tracer does not have that problem *if run paired*: same seed, same data order, same everything, only $\delta$ differs, then difference the two runs. Seed variance cancels almost entirely. Three paired seeds will resolve effects that fifteen unpaired ones cannot.

**Carry a sign, not a number.** What has to be stable across axes is the *sign of $\gamma$* and the *ordering between LSD and PSD* — not $\gamma$'s value. Establishing a stable sign needs 2–3 levels per axis; establishing a stable magnitude would need many more. Written as "PSD's transfer coefficient is positive and below LSD's, across $d$, data regime, and conditioning scheme", the design shrinks accordingly.

## The vehicle for the dimensionality axis

Do not try to scale the checkerboard — the cell count blows up. Use a **Gaussian mixture target**, which keeps the oracle exact at any $d$:

With $x_1 \sim \sum_k \pi_k N(\mu_k,\Sigma_k)$, $x_0\sim N(0,\sigma^2 I)$, linear interpolant: conditioned on component $k$, $I_t \sim N(t\mu_k,\; t^2\Sigma_k+(1-t)^2\sigma^2 I)$. So $p_t$ is a GMM with the same weights at every $t$, the responsibilities are closed-form, each within-component conditional is linear-Gaussian, and

$$
b_t(x)=\frac{\mathbb E[x_1\mid I_t=x]-x}{1-t}
$$

is exact in any dimension, any number of modes. Curvature is tunable via mode separation and $\Sigma_k$ — which means **the geometry axis and the dimensionality axis run on the same generator**, and the near-affine positive control is just $M=1$. Keep the checker as the centre cell for continuity with what has already run.

So the oracle is not a 2-D-only luxury. It scales with the study.

## What actually transfers to CIFAR

The reason images were under-weighted in Part 1 was that $\gamma$ appeared to need ground truth. It does not:

$$
\gamma(s,t;\lambda) = \frac{\langle \hat X_{s,t}-X^{\mathrm{ref}}_{s,t},\; X^{\delta}_{s,t}-X^{\mathrm{ref}}_{s,t}\rangle}{\lVert X^{\delta}_{s,t}-X^{\mathrm{ref}}_{s,t}\rVert^2}
$$

Both reference maps come from integrating a velocity field already in hand — a well-trained FM-only teacher, and that same teacher plus the injected $\delta$. Neither has to be the true field. $\delta$ is known exactly by construction, so the transport measurement is valid wherever a teacher can be trained. What is lost on images is distance-to-truth, not the causal metric.

Practical consequence: CIFAR does not need a replicated η sweep. It needs **the tracer at 2–3 injection strengths, 2 families, 2 paired seeds** — roughly 12 runs at image scale, not 70. That is the difference between "we scaled" being affordable and not.

## Budget

| Stage | Cells | Runs | Gate to proceed |
| --- | --- | --- | --- |
| Centre cell: E1 tracer, checker $d{=}2$ | 1 | ~20 paired | $\gamma$ separable from zero for PSD |
| Tier-1 star: $d$, time-conditioning, data regime | +4 | ~32 paired | Sign stable on all three |
| Tier-2 star: geometry, coupling, capacity | +4 | ~32 paired | Only if Tier 1 holds; geometry doubles as the measurement control |
| CIFAR-10 tracer | 1 | ~12 | Ships the generality claim |

≈ 96 runs, comparable to the 70 already spent, and every one answers a named objection.

## Bottom line on "how many things"

**Five axes at 2–3 levels each, run as a star with paired seeds, plus one image dataset.** Not because five is a magic number, but because that is exactly how many entries are on the adversarial list — and when a reviewer names a sixth, a sixth arm gets added, the study does not restart.
