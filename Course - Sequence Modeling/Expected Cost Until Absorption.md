# Expected Cost Until Absorption

Tags: #markov-chain #absorbing #cost #bellman

---

## Setup

In an [[Absorbing Markov Chain]], each transition $(j, k)$ from a transient state has an associated non-negative cost $c_{jk} \geq 0$.

We want to compute $V(i)$: the **expected total cost until absorption** when starting from transient state $i$.

---

## Bellman-Style Equation

$$V(i) = \sum_{k \in \text{Succ}(i)} p_{ik}(c_{ik} + V(k))$$

with boundary conditions:
$$V(a) = 0 \quad \forall a \in \mathcal{A} \text{ (absorbing states)}$$

This is a system of linear equations — one per transient state.

**Intuition:** The expected cost from $i$ equals the expected immediate cost of the next step plus the expected future cost from the new state.

---

## Alternative Formula via Fundamental Matrix

The expected number of transitions through edge $(j,k)$ starting from $i$ is $n_{ij} \cdot p_{jk}$.

Therefore the total expected cost:

$$V(i) = \sum_{(j,k) \in E} n_{ij} \, p_{jk} \, c_{jk}$$

---

## Derivation (Conditional Expectation)

The derivation uses the **law of total expectation** (tower property):

$$\mathbb{E}_{x,y}[f(x,y)] = \mathbb{E}_x\!\left[\mathbb{E}_y[f(x,y) \mid x]\right]$$

Let $V(i)$ be the **expected total cost until absorption** when starting from transient state $i$. Applied here: condition on the first transition $s_1$, then take the outer expectation — step by step:

$$\begin{align}
V(s_0 = i)
&= \mathbb{E}_{s_1, s_2, \dots}\!\left[\sum_{t=0}^{\infty} c_{s_t s_{t+1}} \;\Bigg|\; s_0 = i\right] \\[8pt]
&= \mathbb{E}_{s_1, s_2, \dots}\!\left[c_{s_0 s_1} + \sum_{t=1}^{\infty} c_{s_t s_{t+1}} \;\Bigg|\; s_0 = i\right] \\[8pt]
&= \mathbb{E}_{s_1}\!\left[\mathbb{E}_{s_2, s_3, \dots}\!\left[c_{s_0 s_1} + \sum_{t=1}^{\infty} c_{s_t s_{t+1}} \;\Bigg|\; s_1\right] \;\Bigg|\; s_0 = i\right] \\[8pt]
&= \mathbb{E}_{s_1}\!\left[c_{s_0 s_1} + \underbrace{\mathbb{E}_{s_2, s_3, \dots}\!\left[\sum_{t=1}^{\infty} c_{s_t s_{t+1}} \;\Bigg|\; s_1\right]}_{V(s_1)} \;\Bigg|\; s_0 = i\right] \\[8pt]
&= \mathbb{E}_{s_1}\!\left[c_{s_0 s_1} + V(s_1) \mid s_0 = i\right] \\[8pt]
&= \sum_{k \in \mathcal{S}\text{ucc}(i)} p_{ik}\bigl(c_{ik} + V(k)\bigr)
\end{align}$$

- Line 1: definition of $V(i)$
- Line 2: peel off the first transition cost $c_{s_0 s_1}$
- Line 3: apply the law of total expectation — condition on $s_1$ first
- Line 4: $c_{s_0 s_1}$ is fixed given $s_1$; the remaining sum is $V(s_1)$ by the **Markov property**
- Line 5: substitute the underbrace
- Line 6: expand the expectation over $s_1$ using transition probabilities $p_{ik}$, where $\mathcal{S}\text{ucc}(i)$ is the **set of successor states of $i$** in the network structure (i.e. all states $k$ such that $p_{ik} > 0$)

---

## Application

Used in [[Application - Customer Lifetime Value]] to compute the expected profit from a customer segment over an infinite horizon.

---

*Part of → [[Absorbing Markov Chain]] | [[Fundamental Matrix]] | [[MOC - Modelling Sequential Data]]*
