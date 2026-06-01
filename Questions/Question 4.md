---
tags:
  - markov
  - markov-chain
  - markov-models
---
>Describe what is a Markov chain and derive in detail (mathematically) its evolution equation providing the evolution of the states vector along time.

---

## Answer

### What is a Markov Chain?

A **Markov chain** is a model of a **sequential, discrete-time, discrete-state stochastic process**. It is characterised by:

- A finite set of states $S = \{1, 2, \dots, n\}$
- $s_t = k$ means the process is in state $k$ at time $t$
- A **transition probability matrix** $\mathbf{P}$ where $p_{ij} = P(s_{t+1}=j \mid s_t=i)$
- Transitions satisfy the **Markov property**

---

### The Markov Property

$$P(s_{t+1} = j \mid s_t = i, s_{t-1}, \dots, s_0) = P(s_{t+1} = j \mid s_t = i) = p_{ij}$$

The future depends **only on the present state**, not on the history. The process is **memoryless**.

---

### Transition Probability Matrix

$\mathbf{P}$ is an $n \times n$ **stochastic matrix**:
- All entries $p_{ij} \geq 0$
- Each row sums to 1: $\sum_j p_{ij} = 1$
- Homogeneous: probabilities do not depend on time $t$

**Example (weather model):** states $\{R, N, S\}$ (Rain, Nice, Snow):

| | **R** | **N** | **S** |
|---|---|---|---|
| **R** | 1/2 | 1/4 | 1/4 |
| **N** | 1/2 | 0 | 1/2 |
| **S** | 1/4 | 1/4 | 1/2 |

---

### Derivation of the Time Evolution Equation

Let $\mathbf{x}(t)$ be the **column vector** of state probabilities at time $t$: $x_i(t) = P(s_t = i)$.

**Derivation of $x_i(t)$:**

$$\begin{align}
x_i(t) &= P(s_t = i) \\[6pt]
&= \sum_{k=1}^{n} P(s_t = i,\ s_{t-1} = k) \\[6pt]
&= \sum_{k=1}^{n} P(s_t = i \mid s_{t-1} = k)\ P(s_{t-1} = k) \\[6pt]
&= \sum_{k=1}^{n} p_{ki}\ x_k(t-1)
\end{align}$$

- Line 2: marginalise over previous state $s_{t-1} = k$
- Line 3: chain rule of probability
- Line 4: Markov property; substitute $p_{ki}$ and $x_k(t-1)$

In matrix form:

$$\boxed{\mathbf{x}(t) = \mathbf{P}^T\, \mathbf{x}(t-1)}$$

Applying recursively from $t=0$:

$$\mathbf{x}(t) = (\mathbf{P}^T)^t\, \mathbf{x}(0)$$

---

### Multi-Step Transition Probabilities

The $\tau$-step transition probability is the $(i,j)$ entry of $\mathbf{P}^\tau$:

$$p_{ij}^{(\tau)} = P(s_{t+\tau}=j \mid s_t=i) = [\mathbf{P}^\tau]_{ij}$$

**Proof for $\tau=2$:**

$$\begin{align}
P(s_{t+2}=j \mid s_t=i)
&= \sum_{k=1}^{n} P(s_{t+2}=j,\ s_{t+1}=k \mid s_t=i) \\
&= \sum_{k=1}^{n} P(s_{t+2}=j \mid s_{t+1}=k)\ P(s_{t+1}=k \mid s_t=i) \\
&= \sum_{k=1}^{n} p_{kj}\ p_{ik} = [\mathbf{P}^2]_{ij}
\end{align}$$

- $\mathbf{P}^2$: two-steps-ahead transition matrix
- $\mathbf{P}^t$: $t$-steps-ahead transition matrix (by induction)

---

### Basis Vector Formulation

When the process starts in state $i$ at $t=0$ (i.e. $\mathbf{x}(0) = \mathbf{e}_i$):

$$x_{j\mid i}(t) = P(s_t=j \mid s_0=i) = \mathbf{e}_i^T\,\mathbf{P}^t\,\mathbf{e}_j$$

This is simply $[\mathbf{P}^t]_{ij}$, consistent with the multi-step formula.

---

### Long-Run Behaviour

For a **regular** Markov chain, $\mathbf{x}(t)$ converges to the unique **stationary distribution** $\boldsymbol{\pi}$ regardless of the initial state:

$$\lim_{t\to\infty} \mathbf{x}(t) = \boldsymbol{\pi}, \quad \text{where } \boldsymbol{\pi} = \mathbf{P}^T\boldsymbol{\pi},\ \sum_i \pi_i = 1$$
