---
tags:
  - markov
  - markov-chain
  - markov-models
  - absorbing
  - question
---
>Describe what is a Markov chain and derive in detail (mathematically) how to compute the expected number of visits to each state before being absorbed in the case of absorbing Markov chains. Also discuss some game/finance application.

---

## Answer

### Markov Chain (brief)

A **Markov chain** is a discrete-time, discrete-state stochastic process satisfying the **Markov property**: the future depends only on the present state, not the history. It is characterised by a transition matrix $\mathbf{P}$ where $p_{ij} = P(s_{t+1}=j\mid s_t=i)$, and a state probability vector evolving as $\mathbf{x}(t) = \mathbf{P}^T\mathbf{x}(t-1)$.

An **absorbing Markov chain** has at least one absorbing state ($p_{aa}=1$). In canonical form:

$$\mathbf{P} = \begin{pmatrix} \mathbf{Q} & \mathbf{R} \\ \mathbf{0} & \mathbf{I} \end{pmatrix}$$

where $\mathbf{Q}$ = transient-to-transient transitions, $\mathbf{R}$ = transient-to-absorbing transitions.

---

### Expected Number Fundamental Matrix

$n_{ij}$ = expected number of times the process visits transient state $j$, starting from transient state $i$, before absorption.

**Key link:** Since $i$ and $j$ are both transient, the conditional probability uses $\mathbf{Q}$ (not $\mathbf{P}$):

$$x_{j\mid i}(t) = P(s_t=j \mid s_0=i) = \mathbf{e}_i^T\mathbf{Q}^t\mathbf{e}_j$$

**Derivation of $n_{ij}$:**

$$\begin{align}
n_{ij} &= \mathbf{e}_i^T\,\mathbf{N}\,\mathbf{e}_j \\[6pt]
&= \mathbf{e}_i^T \left(\sum_{t=0}^{\infty}\mathbf{Q}^t\right)\mathbf{e}_j \\[6pt]
&= \sum_{t=0}^{\infty} \left(\mathbf{e}_i^T\mathbf{Q}^t\mathbf{e}_j\right) \\[6pt]
&= \sum_{t=0}^{\infty} x_{j\mid i}(t)
\end{align}$$

- Line 2: definition of $\mathbf{N} = \sum_{t=0}^{\infty}\mathbf{Q}^t$
- Line 3: linearity — basis vectors pass through the sum
- Line 4: recognise $x_{j\mid i}(t)$

So $n_{ij}$ = **sum over all time steps of the probability of visiting state $j$ at time $t$ starting from $i$** = expected number of visits.

**The Fundamental Matrix:**

$$\boxed{\mathbf{N} = \sum_{t=0}^{\infty}\mathbf{Q}^t = (\mathbf{I}-\mathbf{Q})^{-1}}$$

This sum converges because $\mathbf{Q}^t \to \mathbf{0}$ as $t\to\infty$ (transient states are eventually left).

**Total expected visits before absorption** (row sums of $\mathbf{N}$):

$$\vec{\mathbf{n}} = \mathbf{N}\mathbf{e}$$

Entry $i$ of $\vec{\mathbf{n}}$ = total expected visits to any transient state, starting from state $i$.

---

### Example: Drunkard's Walk

States: transient $\{1,2,3\}$, absorbing $\{0,4\}$.

$$\mathbf{Q} = \begin{pmatrix}0&1/2&0\\1/2&0&1/2\\0&1/2&0\end{pmatrix}, \quad \mathbf{N} = (\mathbf{I}-\mathbf{Q})^{-1} = \begin{pmatrix}3/2&1&1/2\\1&2&1\\1/2&1&3/2\end{pmatrix}$$

$$\mathbf{n} = \mathbf{N}\mathbf{e} = \begin{pmatrix}3\\4\\3\end{pmatrix}$$

Starting from state 2, the drunkard visits transient states **4 times on average** before being absorbed at a wall. Starting from state 1 or 3, only 3 visits on average (closer to a wall).

---

### Game / Finance Applications

**Gambler's ruin:** $n_{ij}$ gives the expected number of rounds spent with wealth $j$, starting from wealth $i$. The total $\vec{n}_i = \sum_j n_{ij}$ gives the expected duration of the game.

**Customer lifecycle:** transient states = active customer segments, absorbing state = "lost customer." $n_{ij}$ = expected number of months a customer spends in segment $j$ before churning, given they start in segment $i$. This feeds directly into the **Customer Lifetime Value** calculation.
