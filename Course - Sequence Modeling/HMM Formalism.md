# HMM Formalism

Tags: #HMM #notation #parameters

---

## Notation

For a [[Hidden Markov Model (HMM)]] of length $T$:

### States
- $s(t) \in \{1, \dots, n\}$: hidden state at time $t$ ($n$ possible states)

### Observations
- $x(t) \in \{o_1, \dots, o_p\}$: observation at time $t$ ($p$ possible values)
- $\mathbf{x} = [x_1, x_2, \dots, x_T]^T$: full observation sequence

### Parameters $\theta = \{\mathbf{\Pi}, \mathbf{P}, \mathbf{B}\}$

**Initial state probabilities:**
$$\pi_i = P(s(1) = i)$$

**State transition probabilities:**
$$p_{ij} = P(s(t+1) = j \mid s(t) = i)$$

**Emission (observation) probabilities:**
$$b_i(o_k) = P(x(t) = o_k \mid s(t) = i)$$

---

## Key Independence Assumptions

1. **[[Markov Property]]**: $P(s_t \mid s_{t-1}, s_{t-2}, \dots) = P(s_t \mid s_{t-1})$
2. **Observation independence**: $P(x_t \mid s_t, s_{t-1}, \dots, x_{t-1}, \dots) = P(x_t \mid s_t)$

---

## Joint Probability of a State-Observation Sequence

$$P(\mathbf{x}, \mathbf{s} \mid \theta) = \pi_{s_1} b_{s_1}(x_1) \prod_{t=1}^{T-1} p_{s_t s_{t+1}} b_{s_{t+1}}(x_{t+1})$$

---

*Part of → [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
