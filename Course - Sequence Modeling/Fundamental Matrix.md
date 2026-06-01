# Fundamental Matrix

Tags: #markov-chain #absorbing #fundamental-matrix

---

## Definition

For an [[Absorbing Markov Chain]] with sub-stochastic matrix $\mathbf{Q}$ (transitions between transient states), the **fundamental matrix** is:

$$\mathbf{N} = \mathbf{I} + \mathbf{Q} + \mathbf{Q}^2 + \mathbf{Q}^3 + \cdots = (\mathbf{I} - \mathbf{Q})^{-1}$$

This infinite sum converges because $\mathbf{Q}^t \to \mathbf{0}$ as $t \to \infty$.

---

## Interpretation

The element $n_{ij} = [\mathbf{N}]_{ij}$ is the **expected number of visits to transient state $j$** when starting from transient state $i$, before absorption.

**Key link to transient probabilities:**

Since $i$ and $j$ are both transient states, the conditional probability $x_{j\mid i}(t)$ uses $\mathbf{Q}$ instead of $\mathbf{P}$:

$$x_{j\mid i}(t) = P(s_t = j \mid s_0 = i) = \mathbf{e}_i^T\, \mathbf{Q}^t\, \mathbf{e}_j$$

This is because transient-to-transient dynamics are governed by $\mathbf{Q}$ alone (the sub-block of $\mathbf{P}$).

**Proof that $n_{ij} = \mathbf{e}_i^T \mathbf{N} \mathbf{e}_j$:**

$$\begin{align}
n_{ij} &= \mathbf{e}_i^T\, \mathbf{N}\, \mathbf{e}_j \\[6pt]
&= \mathbf{e}_i^T \left(\sum_{t=0}^{\infty} \mathbf{Q}^t\right) \mathbf{e}_j \\[6pt]
&= \sum_{t=0}^{\infty} \left(\mathbf{e}_i^T\, \mathbf{Q}^t\, \mathbf{e}_j\right) \\[6pt]
&= \sum_{t=0}^{\infty} x_{j\mid i}(t)
\end{align}$$

- Line 1: definition, entry $(i,j)$ of $\mathbf{N}$ extracted via basis vectors
- Line 2: substitute $\mathbf{N} = \sum_{t=0}^{\infty} \mathbf{Q}^t$
- Line 3: linearity — basis vectors pass through the sum
- Line 4: recognise $\mathbf{e}_i^T \mathbf{Q}^t \mathbf{e}_j = x_{j\mid i}(t)$

So $n_{ij}$ is the **sum over all time steps of the probability of being in state $j$ at time $t$ starting from $i$** — i.e. the **expected number of passages (visits) through transient state $j$, starting from transient state $i$, before being absorbed**.

The **total expected number of visits** across all transient states, for each possible starting state, is given by summing each row of $\mathbf{N}$:

$$\vec{\mathbf{n}} = \mathbf{N}\, \mathbf{e}$$

where $\mathbf{e}$ is the all-ones column vector. Entry $i$ of $\vec{\mathbf{n}}$ gives the total expected number of visits to any transient state before absorption, when starting from transient state $i$.

---

## Total Expected Visits

The vector of **total expected visits before absorption** (summing over all transient states $j$):

$$\mathbf{n} = \mathbf{N}\mathbf{e}$$

where $\mathbf{e}$ is the all-ones vector.

---

## Example (Drunkard's Walk)

States: transient $\{1, 2, 3\}$, absorbing $\{0, 4\}$. The full transition matrix in canonical form (transient block $\mathbf{Q}$, then absorbing states):

$$\mathbf{P} = \begin{pmatrix} 0 & 1/2 & 0 & 1/2 & 0 \\ 1/2 & 0 & 1/2 & 0 & 0 \\ 0 & 1/2 & 0 & 0 & 1/2 \\ 0 & 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 0 & 1 \end{pmatrix} \quad \begin{array}{l} \leftarrow 1 \\ \leftarrow 2 \\ \leftarrow 3 \\ \leftarrow 0 \\ \leftarrow 4 \end{array}$$

The transient sub-block $\mathbf{Q}$ (rows/columns $\{1,2,3\}$):

$$\mathbf{Q} = \begin{pmatrix} 0 & 1/2 & 0 \\ 1/2 & 0 & 1/2 \\ 0 & 1/2 & 0 \end{pmatrix}$$

Fundamental matrix:

$$\mathbf{N} = (\mathbf{I} - \mathbf{Q})^{-1} = \begin{pmatrix} 3/2 & 1 & 1/2 \\ 1 & 2 & 1 \\ 1/2 & 1 & 3/2 \end{pmatrix}$$

Total expected visits before absorption:

$$\mathbf{n} = \mathbf{N}\mathbf{e} = \begin{pmatrix} 3/2 & 1 & 1/2 \\ 1 & 2 & 1 \\ 1/2 & 1 & 3/2 \end{pmatrix} \begin{pmatrix} 1 \\ 1 \\ 1 \end{pmatrix} = \begin{pmatrix} 3 \\ 4 \\ 3 \end{pmatrix}$$

Starting from state 2, the walker visits transient states 4 times on average before being absorbed. By symmetry, starting from states 1 or 3 gives 3 expected visits.

---

## Extensions

The fundamental matrix can also be generalized to regular (non-absorbing) chains to compute **average first-passage times**.

---

*Part of → [[Absorbing Markov Chain]] | [[MOC - Modelling Sequential Data]]*
