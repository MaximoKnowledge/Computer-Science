---
state: read
name: Evaluating the World Model Implicit in a Generative Model
link: https://arxiv.org/abs/2406.03689v3
tldr: Showed that a model can be extremely-accurate at predicting feasible next states while failing to internalise key notions of the World
note:
quality:
  - banger
abstract: "Recent work suggests that large language models may implicitly learn world models. How should we assess this possibility? We formalize this question for the case where the underlying reality is governed by a deterministic finite automaton. This includes problems as diverse as simple logical reasoning, geographic navigation, game-playing, and chemistry. We propose new evaluation metrics for world model recovery inspired by the classic Myhill-Nerode theorem from language theory. We illustrate their utility in three domains: game playing, logic puzzles, and navigation. In all domains, the generative models we consider do well on existing diagnostics for assessing world models, but our evaluation metrics reveal their world models to be far less coherent than they appear. Such incoherence creates fragility: using a generative model to solve related but subtly different tasks can lead to failures. Building generative models that meaningfully capture the underlying logic of the domains they model would be immensely valuable; our results suggest new ways to assess how close a given model is to that goal."
paper id: 2406.03689v3
authors:
  - Keyon Vafa
  - Justin Y. Chen
  - Ashesh Rambachan
  - Jon Kleinberg
  - Sendhil Mullainathan
publication date: 2024-06-06T02:20
comments: ""
pdf: https://arxiv.org/pdf/2406.03689v3
tags:
  - world
  - generative
  - xai
---
#paper
## Takeaways
- Took a theoretical computer science approach to evaluating World models. They focus on NTP models from a vocabulary $\Sigma$. They make one key definition: the Myhill-Nerode boundary. This boundary is the set of all sequences of tokens that were valid in both trajectories of a starting state $q_1$ and another starting state $q_2$ ($q_{1} \neq q_{2}$) up-until the last token, where the trajectory got invalid for the path of $q_{2}$ but not of $q_1$. More formally:
$${ \bf M N B } ^ { W } ( q _ { 1 }, q _ { 2 } ) = \{ s = a _ { 1 } a _ { 2 }... a _ { k } \mid s \in L ^ { W } ( q _ { 1 } ) \setminus L ^ { W } ( q _ { 2 } ) \; { \bf a n d } \; \forall j < k : a _ { 1 }... a _ { j } \in { \bf M N I } ^ { W } ( q _ { 1 }, q _ { 2 } ) \}$$
So, L(q1) is the set of all legal trajectories starting from q1 (same for L(q2)). $\mathrm{MNI}(q_{1},q_{2}):=L(q_{1}) \cap L(q_{2})$. So essentially, the boundary are the first points in which the trajectories of q1 and q2 differ in their set of legal moves. E.g. if we're on the connect 4 scenario (explained later), the boundary would be when we have two games q1 and q2, and on one we've just filled a column while in the other not:
![[Pasted image 20260927201207.png]]
So, in the picture we drop three tokens on to the first column. That invalidates the first column for q2 for all successive moves, but for q1 it's still good to use. Thus that sequence of moves corresponds to the boundary.

Connect 4 game: the rules of the game aren't classic connect-4, but rather it's dropping the tokens so they fall on the bottom. The only rules is that once a column gets filled you can't drop more tokens. Thus, the set of actions is an index over the columns and the state is the current situation of the board (there is no winner nor loser, the idea is just to test a generative model's capability of understanding the constraints and equivalences of the board game scenarios).

- From the MNB metric they define two other metrics: compression and distinction. Basically, the compression metric means that the model should be Markovian when it has two trajectories that are different but lead to the same state q (i.e. the trajectory is not important, the key is the current state); distinction means that for two different states q1 and q2, that are at the boundary, the model gives different predictions. 

- They train three models to perform path traversal through Manhattan, the goal of the models is that given start A and end B to produce a sequence of moves (N, S, W, E, NE, NW, SE, SW) that are legal (so respect the true streets) and eventually arrive to B; the difference between the models is the training data: shortest path between (A,B), shortest path between (A,B) with some detours, completely random walk between (A,B) that arrives to B in 100 steps. They found that the random-walk model respected the most the previously-introduced metrics, while the shortest path failed to either compress or distinguish paths. I didn't go so deep into how they compute the metrics, but they're not the best way to assess whether a model has a good representation of the World honestly.

## I+D
- The training recipe is substantially unfair for the shortest-path model: it sees more data and it's trained until it overfits, and then they pick the model with the best validation error. An analysis of the mechanics of the model during training would be pretty insightful to see when it collapses and what differs between the models.
- Their metric has a big implicit bias in how the models compress and represent the state. Constraining the model's representation and state capacity (e.g. with an LSTM) may fool the metrics with a model that compacts more the information but does not amortise so well to out-of-distribution examples.

## Deep Dive

