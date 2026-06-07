---
tags:
  - markov-chain
  - stochastic-process
  - sequential-data
  - sequence-modeling
---
# Markov Chain

Tags: #markov-chain #stochastic-process #sequential-data

---

## Definition

A **Markov chain** is a model of a **sequential, discrete-time, discrete-state stochastic process**.

- Finite set of states: $S = \{1, 2, \dots, n\}$
- $s_t = k$ means the process is in state $k$ at time $t$
- Transitions between states follow the **[[Markov Property]]**

---

## Transition Probability Matrix

The key object is the matrix $\mathbf{P}$ — see → [[Transition Probability Matrix]]

$$p_{ij} = P(s_{t+1} = j \mid s_t = i)$$

---

## Time Evolution

Let $\mathbf{x}(t)$ be the **column vector** containing the probability distribution over all states at time step $t > 0$, i.e. $x_i(t) = P(s_t = i)$.

**Derivation of $x_i(t)$:**

$$\begin{align}
x_i(t) &= P(s_t = i) \\[6pt]
&= \sum_{k=1}^{n} P(s_t = i,\ s_{t-1} = k) \\[6pt]
&= \sum_{k=1}^{n} P(s_t = i \mid s_{t-1} = k)\ P(s_{t-1} = k) \\[6pt]
&= \sum_{k=1}^{n} p_{ki}\ x_k(t-1)
\end{align}$$

- Line 1: definition of $x_i(t)$
- Line 2: marginalise over previous state $s_{t-1} = k$
- Line 3: chain rule of probability
- Line 4: substitute $p_{ki} = P(s_t = i \mid s_{t-1} = k)$ and $x_k(t-1) = P(s_{t-1} = k)$

In matrix form this gives the **time evolution equation**:

$$\mathbf{x}(t) = \mathbf{P}^T\, \mathbf{x}(t-1) = (\mathbf{P}^T)^t\, \mathbf{x}(0)$$

---

## Multi-Step Probabilities

$$p_{ij}^{(\tau)} = P(s_{t+\tau} = j \mid s_t = i) = [\mathbf{P}^\tau]_{ij}$$

- $\mathbf{P}^2$ = 2-step transition matrix
- $\mathbf{P}^\tau$ = $\tau$-step transition matrix

---

## Long-Run Behaviour

For **[[Irreducible and Regular Markov Chains]]**, the chain converges to a unique **[[Stationary Distribution]]** $\boldsymbol{\pi}$ regardless of initial state:

$$\lim_{t \to \infty} \mathbf{x}(t) = \boldsymbol{\pi}$$

---

## Special Types

| Type | Description | Notes |
|---|---|---|
| Regular | Some power of P has all positive entries | Unique stationary distribution |
| Irreducible | Every state reachable from every state | Strong connectivity |
| Absorbing | Has absorbing states ($p_{ii}=1$) | See [[Absorbing Markov Chain]] |

---

## Example (Weather)

**States:** Rain (R), Nice (N), Snow (S) — the weather on any given day.

The transition matrix $\mathbf{P}$ where $p_{ij} = P(\text{tomorrow} = j \mid \text{today} = i)$:

|       | **R** | **N** | **S** |
| ----- | ----- | ----- | ----- |
| **R** | 1/2   | 1/4   | 1/4   |
| **N** | 1/2   | 0     | 1/2   |
| **S** | 1/4   | 1/4   | 1/2   |

$$\mathbf{P} = \begin{pmatrix} 1/2 & 1/4 & 1/4 \\ 1/2 & 0 & 1/2 \\ 1/4 & 1/4 & 1/2 \end{pmatrix}$$

**Reading the matrix (row = current state, column = next state):**
- From **R**: 50% chance of staying rainy, 25% nice, 25% snow
- From **N**: 50% chance of rain, 0% staying nice, 50% snow
- From **S**: 25% rain, 25% nice, 50% staying snowy

**Properties:**
- Each row sums to 1 ✓
- Nice weather (N) never follows nice weather ($p_{NN} = 0$)
- The chain is **irreducible** (every state is reachable from every other state) → a unique [[Stationary Distribution]] exists

---

*Part of → [[MOC - Modelling Sequential Data]]*
