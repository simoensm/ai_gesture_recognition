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

If $\mathbf{x}(t)$ is the column vector of state probabilities at time $t$:

$$\mathbf{x}(t) = (\mathbf{P}^T)^t \, \mathbf{x}(0)$$

This is the **time evolution equation** of the Markov chain.

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

States: Rain (R), Nice (N), Snow (S)

$$\mathbf{P} = \begin{pmatrix} 1/2 & 1/4 & 1/4 \\ 1/2 & 0 & 1/2 \\ 1/4 & 1/4 & 1/2 \end{pmatrix}$$

---

*Part of → [[MOC - Modelling Sequential Data]]*
