# Fundamental Matrix

Tags: #markov-chain #absorbing #fundamental-matrix

---

## Definition

For an [[Absorbing Markov Chain]] with sub-stochastic matrix $\mathbf{Q}$ (transitions between transient states), the **fundamental matrix** is:

$$\mathbf{N} = \mathbf{I} + \mathbf{Q} + \mathbf{Q}^2 + \mathbf{Q}^3 + \cdots = (\mathbf{I} - \mathbf{Q})^{-1}$$

This infinite sum converges because $\mathbf{Q}^t \to \mathbf{0}$ as $t \to \infty$.

---

## Interpretation

The element $n_{ij} = [\mathbf{N}]_{ij}$ is the **expected number of visits to transient state $j$** when starting from transient state $i$, before absorption.

**Proof:**
$$n_{ij} = \sum_{t=0}^{\infty} x_{j|i}(t) = \sum_{t=0}^{\infty} [\mathbf{Q}^t]_{ij}$$

where $x_{j|i}(t) = P(s_t = j \mid s_0 = i)$ is the probability of being in state $j$ at time $t$.

---

## Total Expected Visits

The vector of **total expected visits before absorption** (summing over all transient states $j$):

$$\mathbf{n} = \mathbf{N}\mathbf{e}$$

where $\mathbf{e}$ is the all-ones vector.

---

## Example (Drunkard's Walk)

$$\mathbf{Q} = \begin{pmatrix} 0 & 1/2 & 0 \\ 1/2 & 0 & 1/2 \\ 0 & 1/2 & 0 \end{pmatrix}$$

$$\mathbf{N} = (\mathbf{I} - \mathbf{Q})^{-1} = \begin{pmatrix} 3/2 & 1 & 1/2 \\ 1 & 2 & 1 \\ 1/2 & 1 & 3/2 \end{pmatrix}$$

$$\mathbf{n} = \mathbf{N}\mathbf{e} = \begin{pmatrix} 3 \\ 4 \\ 3 \end{pmatrix}$$

Starting from state 2, the walker visits transient states 4 times on average before being absorbed.

---

## Extensions

The fundamental matrix can also be generalized to regular (non-absorbing) chains to compute **average first-passage times**.

---

*Part of → [[Absorbing Markov Chain]] | [[MOC - Modelling Sequential Data]]*
