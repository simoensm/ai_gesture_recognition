Tags: #markov-chain #matrix #stochastic

---

## Definition

The **transition probability matrix** $\mathbf{P}$ of a [[Markov Chain]] is an $n \times n$ matrix where:

$$p_{ij} = P(s_{t+1} = j \mid s_t = i)$$

---

## Properties

- **Stochastic matrix**: all entries $\geq 0$ and each row sums to 1
- Encodes all one-step transition probabilities
- **Homogeneous**: probabilities are assumed **independent of time**

---

## Multi-Step Transitions

$$p_{ij}^{(\tau)} = P(s_{t+\tau} = j \mid s_t = i) = [\mathbf{P}^\tau]_{ij}$$

**Proof** (for 2 steps):

$$P(s_{t+2} = j \mid s_t = i) = \sum_{k=1}^n p_{kj} p_{ik} = [\mathbf{P}^2]_{ij}$$

By induction: $[\mathbf{P}^\tau]_{ij}$ gives $\tau$-step probabilities.

---

## Canonical Form (for Absorbing Chains)

When the chain has absorbing states, $\mathbf{P}$ can be written in **canonical form**:

$$\mathbf{P} = \begin{pmatrix} \mathbf{Q} & \mathbf{R} \\ \mathbf{0} & \mathbf{I} \end{pmatrix}$$

- $\mathbf{Q}$: transitions between **transient** states (sub-stochastic)
- $\mathbf{R}$: transitions from **transient** to **absorbing** states
- Since $\mathbf{Q}$ is sub-stochastic: $\mathbf{Q}^t \to \mathbf{0}$ as $t \to \infty$

See → [[Absorbing Markov Chain]], [[Fundamental Matrix]]

---

*Part of → [[Markov Chain]] | [[MOC - Modelling Sequential Data]]*
