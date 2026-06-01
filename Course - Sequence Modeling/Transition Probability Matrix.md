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

$$\begin{align}
P(s_{t+2} = j \mid s_t = i)
&= \sum_{k=1}^{n} P(s_{t+2} = j,\ s_{t+1} = k \mid s_t = i) \\[6pt]
&= \sum_{k=1}^{n} P(s_{t+2} = j \mid s_t = i,\ s_{t+1} = k)\ P(s_{t+1} = k \mid s_t = i) \\[6pt]
&= \sum_{k=1}^{n} P(s_{t+2} = j \mid s_{t+1} = k)\ P(s_{t+1} = k \mid s_t = i) \\[6pt]
&= \sum_{k=1}^{n} p_{kj}\ p_{ik} \\[6pt]
&= [\mathbf{P}^2]_{ij}
\end{align}$$

- Line 1: marginalise over intermediate state $s_{t+1} = k$
- Line 2: chain rule of probability
- Line 3: [[Markov Property]] — future depends only on the present
- Line 4: substitute $p_{ij} = P(s_{t+1}=j \mid s_t=i)$
- Line 5: definition of matrix multiplication

### Basis Vector Formulation

Using the time evolution equation $\mathbf{x}(t) = (\mathbf{P}^T)^t\, \mathbf{x}(0)$, the probability of being in state $j$ at time $t$ can be written as:

$$x_j(t) = \mathbf{x}(t)^T \mathbf{e}_j = \mathbf{x}(0)^T \mathbf{P}^t\, \mathbf{e}_j$$

where $\mathbf{e}_j$ is the $j$-th **canonical basis vector** (zero everywhere except a $+1$ at position $j$).

**Conditional on a fixed starting state:** when the process starts in state $i$ at $t = 0$, we have $\mathbf{x}(0) = \mathbf{e}_i$, and thus:

$$x_{j \mid i}(t) = P(s_t = j \mid s_0 = i) = \mathbf{e}_i^T\, \mathbf{P}^t\, \mathbf{e}_j = \mathbf{e}_j^T\, (\mathbf{P}^T)^t\, \mathbf{e}_i$$

This defines $x_{j\mid i}(t)$: the probability of observing the process in state $j$ at time $t$, given that it started in state $i$ at $t = 0$. Note that this is simply the $(i,j)$ entry of $\mathbf{P}^t$, consistent with the multi-step formula $p_{ij}^{(t)} = [\mathbf{P}^t]_{ij}$.

**Interpretation:**
- $\mathbf{P}^2$ is the **two-steps-ahead transition probability matrix**: entry $[\mathbf{P}^2]_{ij}$ gives the probability of going from state $i$ to state $j$ in exactly 2 steps.
- By induction, $\mathbf{P}^t$ is the **$t$-steps-ahead transition probability matrix**: entry $[\mathbf{P}^t]_{ij} = P(s_{t+\tau} = j \mid s_t = i)$ gives the probability of reaching state $j$ from state $i$ in exactly $t$ steps.

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
