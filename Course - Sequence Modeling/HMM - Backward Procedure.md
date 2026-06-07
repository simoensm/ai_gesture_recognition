---
tags:
  - HMM
  - backward
  - dynamic-programming
  - beta
  - sequence-modeling
---
# HMM - Backward Procedure

Tags: #HMM #backward #dynamic-programming #beta

---

## Definition

The **backward variable** $\beta_i(t)$ is the probability of observing the **future** observations $x_{t+1}, \dots, x_T$, given that the system is in state $i$ at time $t$:

$$\beta_i(t) = P(x_{t+1} \dots x_T \mid s_t = i, \theta)$$

---

## Recurrence

**Initialization** ($t = T$):
$$\beta_i(T) = 1 \quad \forall i$$

**Induction** ($t = T-1, \dots, 1$):
$$\beta_i(t) = \sum_{j=1}^n p_{ij} \cdot b_j(x_{t+1}) \cdot \beta_j(t+1)$$

---

## Derivation

### General case (induction step)

$$\begin{align}
\beta_i(t) &= P(x_{t+1} \dots x_T \mid s_t = i) \\[6pt]
&= \sum_{j=1}^{n} P(x_{t+1} \dots x_T,\ s_{t+1} = j \mid s_t = i) \\[6pt]
&= \sum_{j=1}^{n} P(x_{t+1} \mid s_{t+1} = j)\ P(s_{t+1} = j \mid s_t = i)\ P(x_{t+2} \dots x_T \mid s_{t+1} = j) \\[6pt]
&= \sum_{j=1}^{n} p_{ij}\ b_j(x_{t+1})\ \beta_j(t+1)
\end{align}$$

- Line 2: marginalise over $s_{t+1} = j$
- Line 3: chain rule + conditional independence of $x_{t+1}$ given $s_{t+1}$, and Markov property
- Line 4: substitute $p_{ij}$, $b_j(x_{t+1})$, and recognise $\beta_j(t+1)$

### Verification at $t = T-1$

When $t = T-1$ (the last induction step, where $\beta_j(T) = 1$):

$$\begin{align}
\beta_i(T-1) &= P(x_T \mid s_{T-1} = i) \\[6pt]
&= \sum_{j=1}^{n} P(x_T,\ s_T = j \mid s_{T-1} = i) \\[6pt]
&= \sum_{j=1}^{n} P(x_T \mid s_{T-1} = i,\ s_T = j)\ P(s_T = j \mid s_{T-1} = i) \\[6pt]
&= \sum_{j=1}^{n} p_{ij}\ b_j(x_T)
\end{align}$$

But from the general recurrence formula applied at $t = T-1$, we also have:

$$\beta_i(T-1) = \sum_{j=1}^{n} p_{ij}\ b_j(x_T)\ \beta_j(T)$$

Comparing the two expressions for $\beta_i(T-1)$, both must be equal:

$$\sum_{j=1}^{n} p_{ij}\ b_j(x_T) = \sum_{j=1}^{n} p_{ij}\ b_j(x_T)\ \beta_j(T)$$

This holds if and only if $\beta_j(T) = 1\ \forall j$, which **justifies the initialisation** $\beta_i(T) = 1$.

---

## Usage

**Likelihood** (alternative computation):
$$P(\mathbf{x} \mid \theta) = \sum_i \pi_i \beta_i(1)$$

**Combined with forward variable:**
$$P(\mathbf{x}, s_t = i \mid \theta) = \alpha_i(t) \beta_i(t)$$

This is central to [[HMM - Parameter Estimation (Baum-Welch)]].

---

*Part of → [[HMM - Likelihood Computation]] | [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
