# Absorption Probabilities

Tags: #markov-chain #absorbing #probability

---

## Definition

For an [[Absorbing Markov Chain]], the **absorption probability** $b_{ij}$ is the probability of being absorbed by absorbing state $j$, given that we started in transient state $i$.

Since $i$ is a transient state and $j$ is an absorbing state, the probability of reaching $j$ at time $t+1$ is given by the probability of visiting any transient state $k$ at time $t$, then jumping from $k$ to $j$. The total absorption probability is the sum over all possible time steps.

**Full derivation of the formula:**

$$\begin{align}
b_{ij} &= \sum_{t=0}^{\infty} P(s_t \in \mathcal{T},\ s_{t+1} = j \mid s_0 = i) \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} P(s_t = k,\ s_{t+1} = j \mid s_0 = i) \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} P(s_{t+1} = j \mid s_t = k,\ s_0 = i)\ P(s_t = k \mid s_0 = i) \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} \underbrace{P(s_{t+1} = j \mid s_t = k)}_{r_{kj}} \underbrace{P(s_t = k \mid s_0 = i)}_{x_{k\mid i}(t)} \\[6pt]
&= \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} x_{k\mid i}(t)\ r_{kj}
\end{align}$$

- Line 1: $b_{ij}$ is the probability of ever transitioning into $j$ from some transient state, summed over all time steps
- Line 2: marginalise over all transient states $k \in \mathcal{T}$
- Line 3: chain rule of probability
- Line 4: Markov property ($s_{t+1}$ depends only on $s_t$, not $s_0$); identify $r_{kj}$ and $x_{k\mid i}(t)$
- Line 5: final formula

where:
- $\mathcal{T}$ is the **set of transient states** (the sum over $k$ is restricted to $\mathcal{T}$ only)
- $x_{k\mid i}(t) = P(s_t = k \mid s_0 = i)$ is the probability of being in transient state $k$ at time $t$, starting from $i$
- $r_{kj} = P(s_{t+1} = j \mid s_t = k)$ is the transition probability from transient state $k$ to absorbing state $j$

---

## Derivation: from the formula to $\mathbf{B} = \mathbf{NR}$

$$\begin{align}
b_{ij} &= \sum_{t=0}^{\infty} \sum_{k \in \mathcal{T}} x_{k\mid i}(t)\ r_{kj} \\[6pt]
&= \sum_{k \in \mathcal{T}} \left[\sum_{t=0}^{\infty} \mathbf{e}_i^T \mathbf{Q}^t \mathbf{e}_k \right] r_{kj} \\[6pt]
&= \sum_{k \in \mathcal{T}} n_{ik}\ r_{kj} \\[6pt]
&= [\mathbf{NR}]_{ij}
\end{align}$$

- Line 1: formula derived above
- Line 2: substitute $x_{k\mid i}(t) = \mathbf{e}_i^T \mathbf{Q}^t \mathbf{e}_k$ (transient-to-transient probability via basis vectors) and swap the sums
- Line 3: recognise $\sum_{t=0}^{\infty} \mathbf{e}_i^T \mathbf{Q}^t \mathbf{e}_k = n_{ik}$, the $(i,k)$ entry of the [[Fundamental Matrix]] $\mathbf{N} = \sum_{t=0}^{\infty} \mathbf{Q}^t$
- Line 4: definition of matrix multiplication

---

## Matrix Form

Gathering all absorption probabilities into a matrix $\mathbf{B}$:

$$\mathbf{B} = \mathbf{N}\mathbf{R}$$

where:
- $\mathbf{N} = (\mathbf{I} - \mathbf{Q})^{-1}$ is the [[Fundamental Matrix]]
- $\mathbf{R}$ is the sub-matrix of transitions from transient to absorbing states

---

## Interpretation

$b_{ij}$ = the probability that, starting from transient state $i$, the chain eventually gets absorbed at state $j$.

Note: $\sum_j b_{ij} = 1$ for all transient states $i$ (absorption is certain).

---

## Example (Drunkard's Walk)

Transient states $\{1,2,3\}$, absorbing states $\{0, 4\}$.

**Sub-matrix $\mathbf{R}$** (transitions from transient to absorbing states):

| | **0** | **4** |
|---|---|---|
| **1** | 1/2 | 0 |
| **2** | 0 | 0 |
| **3** | 0 | 1/2 |

$$\mathbf{R} = \begin{pmatrix} 1/2 & 0 \\ 0 & 0 \\ 0 & 1/2 \end{pmatrix}$$

**Absorption probability matrix $\mathbf{B} = \mathbf{NR}$:**

$$\mathbf{B} = \mathbf{NR} = \begin{pmatrix} 3/2 & 1 & 1/2 \\ 1 & 2 & 1 \\ 1/2 & 1 & 3/2 \end{pmatrix} \begin{pmatrix} 1/2 & 0 \\ 0 & 0 \\ 0 & 1/2 \end{pmatrix} = \begin{pmatrix} 3/4 & 1/4 \\ 1/2 & 1/2 \\ 1/4 & 3/4 \end{pmatrix}$$

| | **absorbed at 0** | **absorbed at 4** |
|---|---|---|
| **start at 1** | 3/4 | 1/4 |
| **start at 2** | 1/2 | 1/2 |
| **start at 3** | 1/4 | 3/4 |

**Interpretation:** Starting from state 1, there is a 75% chance of being absorbed at 0 and 25% at 4. By symmetry, the probabilities are reversed when starting from state 3. Starting from state 2 (the middle), both absorbing states are equally likely. Note that each row sums to 1 — absorption is certain.

---

*Part of → [[Absorbing Markov Chain]] | [[Fundamental Matrix]] | [[MOC - Modelling Sequential Data]]*
