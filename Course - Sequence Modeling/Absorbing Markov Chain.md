---
tags:
  - markov-chain
  - absorbing
  - transient
  - sequence-modeling
---
# Absorbing Markov Chain

Tags: #markov-chain #absorbing #transient

---

## Definitions

**Absorbing state**: a state $i$ where $p_{ii} = 1$ (impossible to leave).

**Absorbing Markov chain**: a [[Markov Chain]] that contains at least one absorbing state.

**Transient state**: any non-absorbing state. The chain will eventually leave it.

---

## Canonical Form of P

$$\mathbf{P} = \begin{array}{cc} & \begin{array}{cc} \text{TR} & \text{ABS} \end{array} \\ \begin{array}{c} \text{TR} \\ \text{ABS} \end{array} & \begin{pmatrix} \mathbf{Q} & \mathbf{R} \\ \mathbf{0} & \mathbf{I} \end{pmatrix} \end{array}$$

- $\mathbf{Q}$: transitions between transient states (sub-stochastic, row sums $\leq 1$)
- $\mathbf{R}$: transitions from transient to absorbing states
- Since $\mathbf{Q}$ is sub-stochastic: $\mathbf{Q}^t \to \mathbf{0}$ as $t \to \infty$

---

## Power of P

$$\mathbf{P}^t = \begin{pmatrix} \mathbf{Q}^t & \cdots \\ \mathbf{0} & \mathbf{I} \end{pmatrix}$$
Since **Q** is sub-stochastic, it can be shown that :
$$\mathbf{Q}^t \rightarrow{0} \;as\; \mathbf{t} \rightarrow{\infty}  $$
---

## Key Derived Quantities

| Quantity | Formula | Interpretation |
|---|---|---|
| [[Fundamental Matrix]] | $\mathbf{N} = (\mathbf{I} - \mathbf{Q})^{-1}$ | Expected visits to transient states |
| Expected visits vector | $\mathbf{n} = \mathbf{N}\mathbf{e}$ | Total visits before absorption |
| [[Absorption Probabilities]] | $\mathbf{B} = \mathbf{N}\mathbf{R}$ | Probability of reaching each absorbing state |
| [[Expected Cost Until Absorption]] | $V(i) = \sum_k p_{ik}(c_{ik} + V(k))$ | Total cost before absorption |

---

## Example: Drunkard's Walk

States 0 and 4 are absorbing (barriers). States 1, 2, 3 are transient.

$$\mathbf{P} = \begin{pmatrix} 1 & 0 & 0 & 0 & 0 \\ 1/2 & 0 & 1/2 & 0 & 0 \\ 0 & 1/2 & 0 & 1/2 & 0 \\ 0 & 0 & 1/2 & 0 & 1/2 \\ 0 & 0 & 0 & 0 & 1 \end{pmatrix}$$

---

*Part of → [[Markov Chain]] | [[MOC - Modelling Sequential Data]]*
