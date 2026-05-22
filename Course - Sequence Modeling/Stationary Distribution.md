# Stationary Distribution

Tags: #markov-chain #stationary #eigenvector #long-run

---

## Definition

The **stationary distribution** $\boldsymbol{\pi}$ of a [[Markov Chain]] is the probability distribution over states that the chain converges to in the long run, regardless of initial state.

$$\lim_{t \to \infty} \mathbf{x}(t) = \boldsymbol{\pi}$$

---

## Existence Condition

A unique stationary distribution exists if the chain is **[[Irreducible and Regular Markov Chains|regular]]**.

For a regular chain:
$$\lim_{t \to \infty} \mathbf{P}^t = \begin{pmatrix} \boldsymbol{\pi}^T \\ \boldsymbol{\pi}^T \\ \vdots \\ \boldsymbol{\pi}^T \end{pmatrix}$$

All rows converge to the same vector $\boldsymbol{\pi}^T$ → **the system becomes memoryless** over time.

---

## How to Compute It

$\boldsymbol{\pi}$ is the **left eigenvector** of $\mathbf{P}$ corresponding to eigenvalue 1, normalized to sum to 1:

$$\boldsymbol{\pi} = \mathbf{P}^T \boldsymbol{\pi}, \quad \sum_i \pi_i = 1$$

**Derivation:**
$$\boldsymbol{\pi} = \lim_{t \to \infty} \mathbf{x}(t) = \lim_{t \to \infty} \mathbf{P}^T \mathbf{x}(t-1) = \mathbf{P}^T \boldsymbol{\pi}$$

---

## Interpretation

$\pi_i$ = the long-run probability of finding the process in state $i$.

Useful in:
- [[Application - Customer Lifetime Value]] (long-run customer behaviour)
- [[Application - ItemRank (Collaborative Filtering)]] (steady-state ranking)

---

*Part of → [[Markov Chain]] | [[MOC - Modelling Sequential Data]]*
