# Absorption Probabilities

Tags: #markov-chain #absorbing #probability

---

## Definition

For an [[Absorbing Markov Chain]], the **absorption probability** $b_{ij}$ is the probability of being absorbed by absorbing state $j$, given that we started in transient state $i$:

$$b_{ij} = \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} x_{k|i}(t) \, r_{kj}$$

where $\mathcal{T}$ is the set of transient states and $r_{kj}$ is the transition probability from transient state $k$ to absorbing state $j$.

---

## Matrix Form

The absorption probabilities are gathered in the matrix $\mathbf{B}$:

$$\mathbf{B} = \mathbf{N}\mathbf{R}$$

where:
- $\mathbf{N}$ = [[Fundamental Matrix]] $= (\mathbf{I} - \mathbf{Q})^{-1}$
- $\mathbf{R}$ = sub-matrix of transitions from transient to absorbing states

---

## Derivation

$$b_{ij} = \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} x_{k|i}(t) \, r_{kj} = \sum_{k \in \mathcal{T}} n_{ik} r_{kj} = [\mathbf{NR}]_{ij}$$

---

## Interpretation

$b_{ij}$ = the probability that, starting from transient state $i$, the chain eventually gets absorbed at state $j$.

Note: $\sum_j b_{ij} = 1$ for all transient states $i$ (absorption is certain).

---

## Example (Drunkard's Walk)

$$\mathbf{R} = \begin{pmatrix} 1/2 & 0 \\ 0 & 0 \\ 0 & 1/2 \end{pmatrix}, \quad
\mathbf{B} = \mathbf{NR} = \begin{pmatrix} 3/4 & 1/4 \\ 1/2 & 1/2 \\ 1/4 & 3/4 \end{pmatrix}$$

Starting from state 1: 75% chance of being absorbed at 0, 25% at 4.

---

*Part of → [[Absorbing Markov Chain]] | [[Fundamental Matrix]] | [[MOC - Modelling Sequential Data]]*
