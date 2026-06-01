---
tags:
  - markov
  - markov-chain
  - markov-models
  - absorbing
---
>Describe what is a Markov chain and derive in detail (mathematically) how to compute the expected cost before being absorbed by solving a system of linear equations, in the case of absorbing Markov chains. Also discuss some game/finance application.

---

## Answer

### Markov Chain (brief)

A **Markov chain** is a discrete-time, discrete-state stochastic process where the future depends only on the present state (**Markov property**). Defined by $\mathbf{P}$ where $p_{ij}=P(s_{t+1}=j\mid s_t=i)$, evolving as $\mathbf{x}(t) = \mathbf{P}^T\mathbf{x}(t-1)$.

An **absorbing Markov chain** has at least one absorbing state ($p_{aa}=1$). Transient states are eventually left. In canonical form: $\mathbf{P} = \begin{pmatrix}\mathbf{Q}&\mathbf{R}\\\mathbf{0}&\mathbf{I}\end{pmatrix}$.

---

### Setup: Expected Cost Until Absorption

Each transition $(j,k)$ from a transient state carries a non-negative cost $c_{jk} \geq 0$. We want $V(i)$: the **expected total cost until absorption** starting from transient state $i$, with $V(a) = 0$ for all absorbing states $a$.

---

### Derivation via Conditional Expectation

Using the **law of total expectation**: $\mathbb{E}_{x,y}[f(x,y)] = \mathbb{E}_x[\mathbb{E}_y[f(x,y)\mid x]]$.

Condition on the first transition $s_1$:

$$\begin{align}
V(s_0=i)
&= \mathbb{E}\!\left[\sum_{t=0}^{\infty}c_{s_t s_{t+1}}\,\bigg|\,s_0=i\right] \\[6pt]
&= \mathbb{E}_{s_1,s_2,\dots}\!\left[c_{s_0 s_1}+\sum_{t=1}^{\infty}c_{s_t s_{t+1}}\,\bigg|\,s_0=i\right] \\[6pt]
&= \mathbb{E}_{s_1}\!\left[\mathbb{E}_{s_2,s_3,\dots}\!\left[c_{s_0 s_1}+\sum_{t=1}^{\infty}c_{s_t s_{t+1}}\,\bigg|\,s_1\right]\,\bigg|\,s_0=i\right] \\[6pt]
&= \mathbb{E}_{s_1}\!\left[c_{s_0 s_1}+\underbrace{\mathbb{E}_{s_2,s_3,\dots}\!\left[\sum_{t=1}^{\infty}c_{s_t s_{t+1}}\,\bigg|\,s_1\right]}_{V(s_1)}\,\bigg|\,s_0=i\right] \\[6pt]
&= \mathbb{E}_{s_1}\!\left[c_{s_0 s_1}+V(s_1)\mid s_0=i\right] \\[6pt]
&= \sum_{k\in\mathcal{S}\text{ucc}(i)} p_{ik}\bigl(c_{ik}+V(k)\bigr)
\end{align}$$

- Line 2: peel off the first cost $c_{s_0 s_1}$
- Line 3: apply law of total expectation — condition on $s_1$
- Line 4: $c_{s_0 s_1}$ is fixed given $s_1$; future cost from $s_1$ is $V(s_1)$ by **Markov property**
- Line 5: substitute $V(s_1)$
- Line 6: expand the expectation over $s_1$ using transition probabilities

where $\mathcal{S}\text{ucc}(i)$ = set of successor states of $i$ (all $k$ with $p_{ik}>0$).

**Bellman equation:**

$$\boxed{V(i) = \sum_{k\in\mathcal{S}\text{ucc}(i)} p_{ik}\bigl(c_{ik}+V(k)\bigr)}$$

---

### System of Linear Equations

Expanding:

$$V(i) = \sum_k p_{ik}c_{ik} + \sum_k p_{ik}V(k)$$

This gives one equation per transient state. In matrix form (letting $\mathbf{c}_i = \sum_k p_{ik}c_{ik}$):

$$\mathbf{V} = \mathbf{c} + \mathbf{Q}\mathbf{V} \implies (\mathbf{I}-\mathbf{Q})\mathbf{V} = \mathbf{c} \implies \mathbf{V} = (\mathbf{I}-\mathbf{Q})^{-1}\mathbf{c} = \mathbf{N}\mathbf{c}$$

where $\mathbf{N} = (\mathbf{I}-\mathbf{Q})^{-1}$ is the **Fundamental Matrix**.

**Alternative formula via fundamental matrix:**

$$V(i) = \sum_{(j,k)\in E} n_{ij}\,p_{jk}\,c_{jk}$$

The expected number of transitions through edge $(j,k)$ starting from $i$ is $n_{ij}\cdot p_{jk}$, each costing $c_{jk}$.

---

### Game / Finance Applications

**Gambler's ruin:** each round costs 1€ in time (or has some monetary cost). $V(i)$ = expected total cost (e.g. duration or money lost) before the game ends, starting with $i$ euros.

**Customer Lifetime Value (CLV):** states = customer segments, absorbing state = "lost customer," cost $c_{jk}$ = negative of profit earned during transition (or $-m_j$ per period in state $j$). Then $-V(i)$ = expected total profit (lifetime value) from a customer starting in segment $i$.

In the marketing model with discount factor $\gamma$:

$$\mu = \mathbf{x}^T(0)(\mathbf{I}-\gamma\mathbf{P})^{-1}\mathbf{m}$$

where $\mathbf{m}$ = profit vector per state — a direct generalisation of the same linear system.
