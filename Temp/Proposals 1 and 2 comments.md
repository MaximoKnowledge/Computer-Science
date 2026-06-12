## Proposal 1 — improve the idea

- Specify the training algorithm (backprop-through-sampler vs REINFORCE vs adjoint/value-matching). This is the single biggest hole; feasibility hinges on it.
- Decide your parameterization up front: free-form controller $u_φ$, or learn a soft value $V(t,x,e_R)$ and use its gradient (the control-theoretic natural choice). Pick one as primary.
- Add leakage diagnostics — shuffled context labels, swapped embeddings — to prove the controller actually uses $e_R$ rather than learning one generic push.
- Report reward-vs-quality Pareto curves over a $\lambda$ sweep, not point numbers.
- Center the real selling point: after one-time training, a new objective costs K clean evaluations and zero reward queries or gradients at sampling time.
## Proposal 1 — fix the writing

- The "Why this is not classifier guidance / not fine-tuning" sections argue against the weak competitors and read defensively. Reframe as "relationship to," not "why we differ from."
- The "Training Setup" is presented like a contribution but it's standard stochastic optimal control. Present it as adopted foundation. Your honest contribution list is really items 3–4 (reward encoder + held-out evaluation), not 1–5.
- Fix the time convention (reward at $X_{0}$ with a forward $\int_{0}^T$ SDE is inconsistent — state the reverse-time parameterization).
- Watch "concept" creep — these rewards are functions / feature targets, not semantic concepts.

## Proposal 2 — improve the idea

- Cut scope hard. It's 3–5 projects. Commit to one threat model, one representation, one adversary, one hypothesis; everything else is v2.
- Promote the trajectory/attention-space memory to the primary method. The attacks you cite work by making inputs statically benign and harm emergent — which structurally defeats embedding-keyed memory, so your "fallback" is really the main idea.
- Specify the adaptive adversary precisely (knowledge, budget, white-box-to-current-risk-model or not) and show results under both fixed and adaptive adversaries so the gap is visible. This experiment is the whole paper.
- Treat R_unsafe — risk as a terminal-value estimate from a noisy latent — as the crux, not as given. Learning that soft value across all noise levels is the hard subproblem.
- Add a hard benign-retain constraint that gates whether an update is even accepted, not a soft penalty — because overblocking accumulates monotonically while safety gains may saturate.

## Proposal 2 — fix the writing

- It reads as an agenda, not a project: "possible memory components," a giant baseline table, every eval setting listed. Commit to choices instead of enumerating options.
- Delete the "mean unsafe vector" section — it's a strawman; beating it proves nothing and it eats space.
- Drop the "Doob" label or earn it. The rule you wrote (score − $λ\nabla R$) is reward-gradient guidance with a learned head, not an h-transform. The branding doesn't match the math.
- "Existing methods are mostly static" is your load-bearing claim and it's asserted, not argued. State the delta crisply and don't borrow P1's.
- The fallback section quietly concedes the main method may fail — which, again, tells you the fallback should be the main method. Don't let the structure undercut the thesis.
- Carve out the minor-related category explicitly: it should be handled by policy/refusal, never as a "benign neighbor" calibration target or a steering problem. Right now it's listed casually alongside the others, and both reviewers and any ethics check will flag that immediately.
