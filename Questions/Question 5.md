---
tags:
  - absorbing
  - markov
  - markov-chain
  - markov-models
  - question
---
>Describe what is an absorbing Markov chain and derive in detail (mathematically) how to compute the absorbing probabilities (probability of being absorbed in an absorbing state). Also discuss some game/finance application.

---

## Answer

### What is an Absorbing Markov Chain?

An **absorbing Markov chain** is a Markov chain containing at least one **absorbing state**: a state $a$ such that $p_{aa} = 1$ (once entered, never left). All other states are **transient** — they will eventually be left.

For an absorbing chain, absorption is **certain**: starting from any transient state, the process will eventually reach an absorbing state with probability 1.

**Canonical form** of the transition matrix (transient states first, absorbing states last):

$$\mathbf{P} = \begin{pmatrix} \mathbf{Q} & \mathbf{R} \\ \mathbf{0} & \mathbf{I} \end{pmatrix}$$

where:
- $\mathbf{Q}$: transitions between transient states (sub-stochastic, $\mathbf{Q}^t \to \mathbf{0}$)
- $\mathbf{R}$: transitions from transient to absorbing states
- $\mathbf{I}$: absorbing states stay absorbing

---

### Absorption Probabilities

$b_{ij}$ = probability of being absorbed by absorbing state $j$, starting from transient state $i$.

**Derivation:**

The probability of reaching absorbing state $j$ at time step $t+1$, having been in some transient state $k$ at time $t$, summed over all time steps:

$$\begin{align}
b_{ij} &= \sum_{t=0}^{\infty} P(s_t \in \mathcal{T},\ s_{t+1}=j \mid s_0=i) \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k\in\mathcal{T}} P(s_t=k,\ s_{t+1}=j \mid s_0=i) \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k\in\mathcal{T}} P(s_{t+1}=j \mid s_t=k,\ s_0=i)\ P(s_t=k \mid s_0=i) \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k\in\mathcal{T}} \underbrace{P(s_{t+1}=j \mid s_t=k)}_{r_{kj}} \underbrace{P(s_t=k \mid s_0=i)}_{x_{k\mid i}(t)} \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k\in\mathcal{T}} x_{k\mid i}(t)\ r_{kj}
\end{align}$$

- Line 3: chain rule
- Line 4: Markov property + substitution of $r_{kj}$ and $x_{k\mid i}(t)$

where $\mathcal{T}$ = set of transient states (the sum over $k$ is restricted to $\mathcal{T}$ only).

**From formula to matrix form:**

$$\begin{align}
b_{ij} &= \sum_{t=0}^{\infty} \sum_{k\in\mathcal{T}} x_{k\mid i}(t)\ r_{kj} \\[6pt]
&= \sum_{k\in\mathcal{T}} \left[\sum_{t=0}^{\infty} \mathbf{e}_i^T\mathbf{Q}^t\mathbf{e}_k\right] r_{kj} \\[6pt]
&= \sum_{k\in\mathcal{T}} n_{ik}\ r_{kj} \\[6pt]
&= [\mathbf{NR}]_{ij}
\end{align}$$

- Line 2: substitute $x_{k\mid i}(t) = \mathbf{e}_i^T\mathbf{Q}^t\mathbf{e}_k$ and swap sums
- Line 3: recognise $\sum_{t=0}^{\infty}\mathbf{e}_i^T\mathbf{Q}^t\mathbf{e}_k = n_{ik}$, entry of fundamental matrix $\mathbf{N}=(\mathbf{I}-\mathbf{Q})^{-1}$
- Line 4: matrix multiplication

**Result:**

$$\boxed{\mathbf{B} = \mathbf{N}\mathbf{R}}$$

Note: $\sum_j b_{ij} = 1$ for all transient states $i$ — absorption is certain.

---

### Example: Drunkard's Walk

States: transient $\{1,2,3\}$, absorbing $\{0,4\}$. The drunkard moves left or right with equal probability (1/2) and is absorbed at the walls.

$$\mathbf{R} = \begin{pmatrix} 1/2 & 0 \\ 0 & 0 \\ 0 & 1/2 \end{pmatrix}, \quad \mathbf{N} = \begin{pmatrix} 3/2 & 1 & 1/2 \\ 1 & 2 & 1 \\ 1/2 & 1 & 3/2 \end{pmatrix}$$

$$\mathbf{B} = \mathbf{NR} = \begin{pmatrix} 3/4 & 1/4 \\ 1/2 & 1/2 \\ 1/4 & 3/4 \end{pmatrix}$$

| | absorbed at 0 | absorbed at 4 |
|---|---|---|
| start at 1 | 3/4 | 1/4 |
| start at 2 | 1/2 | 1/2 |
| start at 3 | 1/4 | 3/4 |

Starting from state 2 (the middle), both walls are equally likely. By symmetry, starting closer to a wall increases the probability of being absorbed there.

---

### Finance / Game Applications

**Gambler's ruin:** a gambler starts with $k$ euros, bets 1€ each round, and stops when reaching $N$ euros (wins) or 0 euros (ruined). States = wealth $\{0,1,\dots,N\}$, absorbing states = $\{0, N\}$. The absorption probability $b_{k,N}$ gives the probability of winning from wealth $k$.

**Customer churn:** in a customer lifecycle model, "lost customer" is an absorbing state. $b_{ij}$ gives the probability that a customer in segment $i$ is eventually absorbed into the "churned" state vs other outcomes.
