---
tags:
  - markov
  - markov-chain
  - markov-models
  - absorbing
  - question
---
>Describe what is a regular Markov chain as well as its mathematical properties (equilibrium distribution, etc). Also discuss its application to collaborative recommendation (ItemRank).

---

## Answer

### What is a Regular Markov Chain?

A **regular Markov chain** is a Markov chain for which some power $k$ of the transition matrix $\mathbf{P}^k$ has **all strictly positive entries**. This means every state can be reached from every other state in exactly $k$ steps.

- Regular $\Rightarrow$ Irreducible (strongly connected), but not the reverse.
- An **irreducible** chain allows reaching every state from every state (possibly in different numbers of steps).

---

### Key Theorem: Convergence to Stationary Distribution

For a regular Markov chain:

$$\lim_{t\to\infty} \mathbf{P}^t = \begin{pmatrix} \boldsymbol{\pi}^T \\ \boldsymbol{\pi}^T \\ \vdots \\ \boldsymbol{\pi}^T \end{pmatrix}$$

All rows converge to the same vector $\boldsymbol{\pi}^T$. Consequently, regardless of the initial distribution $\mathbf{x}(0)$:

$$\lim_{t\to\infty} \mathbf{x}(t) = \boldsymbol{\pi}$$

The system becomes **memoryless** over time.

---

### Mathematical Properties: Stationary Distribution

The **stationary distribution** $\boldsymbol{\pi}$ is the unique probability vector satisfying:

$$\boldsymbol{\pi} = \mathbf{P}^T\boldsymbol{\pi}, \quad \sum_i \pi_i = 1, \quad \pi_i > 0\ \forall i$$

**Derivation:** Take the limit of the evolution equation $\mathbf{x}(t) = \mathbf{P}^T\mathbf{x}(t-1)$ as $t\to\infty$:

$$\boldsymbol{\pi} = \lim_{t\to\infty}\mathbf{x}(t) = \lim_{t\to\infty}\mathbf{P}^T\mathbf{x}(t-1) = \mathbf{P}^T\boldsymbol{\pi}$$

So $\boldsymbol{\pi}$ is the **left eigenvector** of $\mathbf{P}$ associated with eigenvalue $1$, normalised to sum to 1.

**Interpretation:** $\pi_i$ = long-run fraction of time spent in state $i$.

**Comparison with absorbing chains:**

| Property | Regular Chain | Absorbing Chain |
|---|---|---|
| Long-run behaviour | Converges to $\boldsymbol{\pi}$ | Absorbed with probability 1 |
| Stationary distribution | Unique $\boldsymbol{\pi}$ | Not applicable |
| Key matrix | $\mathbf{P}$ (stochastic) | $\mathbf{N}=(\mathbf{I}-\mathbf{Q})^{-1}$ |

---

### Application: ItemRank (Collaborative Filtering)

**Context:** recommend items to users based on past purchases, using a random walk on a user-item graph.

**Graph structure:**
- Nodes = items (and users in the bipartite version)
- Edges = co-purchase relationships (weighted by frequency)
- Transition matrix $\mathbf{P}$ derived from edge weights

**Random walk with restart** for user $i$:

$$\mathbf{x}_i(k+1) = \alpha\,\mathbf{P}^T\mathbf{x}_i(k) + (1-\alpha)\mathbf{v}_i$$

where:
- $\alpha$ = probability of **continuing** the walk (following a link)
- $(1-\alpha)$ = probability of **restarting** on a previously purchased item
- $\mathbf{v}_i$ = restart distribution: uniform over items purchased by user $i$

**Steady-state solution:**

$$\mathbf{x}_i = (1-\alpha)(\mathbf{I} - \alpha\mathbf{P}^T)^{-1}\mathbf{v}_i$$

Items with the highest entries in $\mathbf{x}_i$ are recommended to user $i$.

**Connection to stationary distribution:** $\mathbf{x}_i$ is a **personalised stationary distribution** (personalised PageRank) — biased toward items already purchased by user $i$. The global (non-personalised) case with uniform restart recovers the standard stationary distribution $\boldsymbol{\pi}$.
