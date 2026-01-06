# The Structural Scavenger Protocol
**A Methodology for Polymathic Research & "Active Re-Discovery"**

## 1. Core Philosophy: Skeleton vs. Skin
To generate novel connections, one must distinguish between the **Instance** (Skin) and the **Dynamic** (Skeleton).

* **The Skin:** The specific domain objects (e.g., Water, Tokens, Money, Cells).
* **The Skeleton:** The mathematical rules governing interaction and evolution (e.g., Flow, Attention, Compounding, Replication).

**The Rule:** Never discard a topic based on its "Skin" (e.g., Fluid Dynamics). Discard it only if it lacks a robust "Skeleton" (e.g., empirical pipe friction coefficients). We search for **Generative Principles**, not **Phenomenological Results**.

---

## 2. The Scrutiny Protocol (The Filter)
To determine if a topic is worth studying, apply this 3-step test to the Table of Contents, Abstract, or Introduction.

### A. The "De-Noun" Operation (Abstraction Test)
Strip the sentence of domain-specific nouns. Replace them with abstract placeholders like [Agent], [State], or [System].

* *Example:* "Predator-Prey Equations for Rabbits and Foxes."
    * *De-Nouned:* "Interaction equations between [Agent A] and [Agent B] where A consumes B."
    * *Verdict:* **Keep.** (Non-linear dynamics; maps to GANs).
* *Example:* "Viscosity values for crude oil in pipelines."
    * *De-Nouned:* "Parameter values for [Specific Fluid] in [Specific Container]."
    * *Verdict:* **Discard.** (Pure data/parameter fitting).

### B. The "Verb" Check (The Dynamic Test)
Look at the mechanism of change. Does the verb imply a trivial operation or a structural constraint?

* **Fitting / Calibrating:** (Discard) $\rightarrow$ Likely just finding a number.
* **Conserving / Preserving:** (Keep) $\rightarrow$ Indicates Symmetry/Invariance (Structure).
* **Minimizing / Maximizing:** (Keep) $\rightarrow$ Indicates Optimization (Compatible with ML).
* **Evolving / Flowing:** (Keep) $\rightarrow$ Indicates Differential Equations (Computation depth).

### C. The "Portability" Test
Ask: *"If I take this equation out of this context, does it die?"*
* Formulas relying on empirical constants (e.g., density of concrete) **Die**.
* Formulas relying on mathematical relationships (e.g., gradients, geometric constraints) **Survive**.

---

## 3. Knowledge Organization: "Dual-Fidelity" System
Avoid over-summarization. Separate the **Storage** (Facts) from the **Linkage** (Insights).

### A. The Storage (High Fidelity)
* **Structure:** Strict categorical folders (`/Physics`, `/Math`, `/ML`).
* **Content:** Pure, technical notes respecting the original domain.
    * *Example:* In `/Physics/Fluids/Navier_Stokes.md`, write about velocity fields and pressure. Do not mention ML.
* **Goal:** Preserve the rigor of the source material for future reference.

### B. The Linkage (The Abstraction Layer)
* **Structure:** A Visual Mindmap.
* **Content:** The "Translation Key" that connects the folders.
    * *Node A:* `[ML] Transformer Attention`
    * *Node B:* `[Math] Graph Laplacian`
    * *The Edge (Link):* "Both measure information flow based on node similarity/proximity."

---

## 4. The Execution Routine: "Neighbor-Node Exploration"
Do not follow "Trends" (New Papers). Follow the **Mathematical Neighborhood**.

1.  **Identify the Skeleton:** Take your current research focus (e.g., Transformers) and identify the core operation (e.g., Weighted Averages / Set Selection).
2.  **Find the Neighbor:** Search for that operation in a different folder (`/Math` or `/Physics`).
    * *Query:* "Mathematical structures for weighted set selection." $\rightarrow$ *Result:* Measure Theory / Spectral Graph Theory.
3.  **Apply Scrutiny:** Run the "De-Noun" test on the new topic. If it passes, study it.
4.  **The Loop:**
    * Study Topic A (ML).
    * Find its Skeleton.
    * Trace Skeleton to Topic B (Math).
    * Study Topic B strictly.
    * *Result:* You naturally discover the "next step" in ML by understanding the constraints of Topic B.