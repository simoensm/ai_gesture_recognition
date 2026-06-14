---
tags:
  - markov
  - markov-chain
  - markov-models
  - absorbing
  - question
---

>Describe and derive in detail (mathematically) how the « lifetime value » of a customer can be computed according to a Markov chain model (marketing application).

---

## Answer

### Model Setup

The customer lifecycle is modelled as an absorbing Markov chain:

- **States** $= \{1, 2, \dots, n\}$: customer segments (e.g. RFM clusters — Recency, Frequency, Monetary value)
- **State $n$** (absorbing): "lost customer" — $p_{nn}=1$, profit = 0
- **Transition probabilities** $p_{ij}$: estimated from observed monthly movements between segments
- **Profit per segment**: $m_i$ (€/month) for a customer in state $i$ (may be negative)
- **Discount factor**: $0 < \gamma \leq 1$ (accounts for time value of money; $\gamma=1$ = no discounting)

---

### Expected Profit on Infinite Horizon

Starting from segment $i$ at $t=0$, the expected **discounted** total profit is:

$$\mu_i = \mathbb{E}\!\left[\sum_{t=0}^{\infty} \gamma^t\, m_{s_t} \,\bigg|\, s_0=i\right]$$

Summing over all states and time steps, using the state probability vector $\mathbf{x}(t)$:

$$\mu = \sum_{t=0}^{\infty} \gamma^t \sum_{i=1}^{n} x_i(t)\,m_i = \sum_{t=0}^{\infty} \gamma^t\,\mathbf{x}^T(t)\,\mathbf{m}$$

---

### Derivation

Substitute the time evolution equation $\mathbf{x}(t) = (\mathbf{P}^T)^t\mathbf{x}(0)$:

$$\mu = \mathbf{x}^T(0)\left(\sum_{t=0}^{\infty} \gamma^t\,\mathbf{P}^t\right)\mathbf{m}$$

The matrix geometric series converges when $\gamma < 1$ (or when absorption is certain):

$$\sum_{t=0}^{\infty} \gamma^t\,\mathbf{P}^t = (\mathbf{I} - \gamma\mathbf{P})^{-1}$$

Therefore:

$$\boxed{\mu = \mathbf{x}^T(0)\,(\mathbf{I} - \gamma\mathbf{P})^{-1}\,\mathbf{m}}$$

**Entry $i$ of $(\mathbf{I}-\gamma\mathbf{P})^{-1}\mathbf{m}$** = expected discounted lifetime value of a customer starting in segment $i$.

---

### Connection to Absorbing Chain Theory

When $\gamma = 1$ and there is a single absorbing state (lost customer, $m_n=0$), the model reduces exactly to the **expected cost until absorption** framework:

$$\mathbf{V} = (\mathbf{I} - \mathbf{Q})^{-1}\mathbf{m}_{|\mathcal{T}} = \mathbf{N}\,\mathbf{m}_{|\mathcal{T}}$$

where $\mathbf{m}_{|\mathcal{T}}$ is the profit vector restricted to transient states, and $\mathbf{N} = (\mathbf{I}-\mathbf{Q})^{-1}$ is the fundamental matrix.

The discounted case ($\gamma < 1$) replaces $\mathbf{Q}$ by $\gamma\mathbf{P}$, which always converges regardless of the chain structure.

---

### Bellman Equation Form

The CLV can also be computed by solving the system:

$$V(i) = m_i + \gamma\sum_{j} p_{ij}\,V(j)$$

with $V(n) = 0$ (lost customer has zero future value). This is one linear equation per segment — the same Bellman structure as expected cost until absorption.

---

### Practical Use

- **Segment prioritisation:** customers in high-CLV segments receive targeted retention offers.
- **Budget allocation:** marketing spend proportional to CLV difference (value of retaining vs losing a customer).
- **Acquisition decisions:** compare CLV of a new customer in segment $i$ to the cost of acquisition.

The key output is a **vector of lifetime values** $\mathbf{V}$, one per segment, computed in closed form by inverting a matrix.
