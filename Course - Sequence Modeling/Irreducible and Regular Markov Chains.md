---
tags:
  - markov-chain
  - convergence
  - ergodic
  - stationary
  - sequence-modeling
---
# Irreducible and Regular Markov Chains

Tags: #markov-chain #convergence #ergodic

---

## Definitions

**Irreducible chain**: it is possible to reach every state from every other state.
→ The chain is "strongly connected."

**Regular chain**: some power $k$ of the [[Transition Probability Matrix]] $\mathbf{P}^k$ has **all positive (non-zero) entries**.
→ Every state can be reached from every state in exactly $k$ steps.

Note: Regular ⟹ Irreducible, but not the reverse.

---

## Key Theorem (Convergence)

For a **regular** [[Markov Chain]]:

$$\lim_{t \to \infty} \mathbf{P}^t = \begin{pmatrix} \boldsymbol{\pi}^T \\ \boldsymbol{\pi}^T \\ \vdots \\ \boldsymbol{\pi}^T \end{pmatrix}$$

All rows converge to the same **[[Stationary Distribution]]** $\boldsymbol{\pi}$.

**Consequences:**
- The limiting distribution is **independent of the initial state**
- The system becomes **memoryless** as $t \to \infty$
- $\boldsymbol{\pi}$ is unique for a regular chain

---

## Comparison with Absorbing Chains

| Property | Regular Chain | [[Absorbing Markov Chain]] |
|---|---|---|
| Long-run behaviour | Converges to $\boldsymbol{\pi}$ | Absorbed with probability 1 |
| Stationary dist. | Unique $\boldsymbol{\pi}$ | Not applicable |
| Key matrix | $\mathbf{P}$ (stochastic) | $\mathbf{N} = (\mathbf{I}-\mathbf{Q})^{-1}$ |

---

*Part of → [[Markov Chain]] | [[Stationary Distribution]] | [[MOC - Modelling Sequential Data]]*
