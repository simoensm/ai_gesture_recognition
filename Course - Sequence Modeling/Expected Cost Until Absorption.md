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

$$V(s_0 = i) = \mathbb{E}_{s_1, s_2, \dots}\left[\sum_{t=0}^{\infty} c_{s_t s_{t+1}} \,\bigg|\, s_0 = i\right]$$

By expanding the first step and using the Markov property:
$$= \sum_{k \in \text{Succ}(i)} p_{ik}(c_{ik} + V(k))$$

---

## Application

Used in [[Application - Customer Lifetime Value]] to compute the expected profit from a customer segment over an infinite horizon.

---

*Part of → [[Absorbing Markov Chain]] | [[Fundamental Matrix]] | [[MOC - Modelling Sequential Data]]*
