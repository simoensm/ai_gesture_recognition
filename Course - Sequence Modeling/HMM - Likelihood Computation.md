---
tags:
  - HMM
  - likelihood
  - forward
  - backward
  - sequence-modeling
---
# HMM - Likelihood Computation

Tags: #HMM #likelihood #forward #backward

---

## Problem

Given an observation sequence $\mathbf{x} = [x_1, \dots, x_T]^T$ and model $\theta = \{\mathbf{\Pi}, \mathbf{P}, \mathbf{B}\}$, compute:
$$P(\mathbf{x} \mid \theta)$$

---

## Decomposition of the Likelihood

To compute $P(\mathbf{x}\mid\theta)$, we first condition on a fixed state sequence $\mathbf{s} = [s_1, \dots, s_T]$, then marginalise over all possible sequences.

**Step 1 — Observation probability given states** (conditional independence of observations):

$$P(\mathbf{x} \mid \mathbf{s}, \theta) = b_{s_1}(x_1)\, b_{s_2}(x_2)\, \cdots\, b_{s_T}(x_T) = \prod_{t=1}^{T} b_{s_t}(x_t)$$

**Step 2 — Probability of a state sequence** (Markov property):

$$P(\mathbf{s} \mid \theta) = \pi_{s_1}\, p_{s_1 s_2}\, p_{s_2 s_3}\, \cdots\, p_{s_{T-1} s_T} = \pi_{s_1} \prod_{t=1}^{T-1} p_{s_t s_{t+1}}$$

**Step 3 — Joint probability** (chain rule):

$$P(\mathbf{x}, \mathbf{s} \mid \theta) = P(\mathbf{x} \mid \mathbf{s}, \theta)\, P(\mathbf{s} \mid \theta)$$

**Step 4 — Marginal likelihood** (sum over all state sequences):

$$P(\mathbf{x} \mid \theta) = \sum_{\mathbf{s}} P(\mathbf{x} \mid \mathbf{s}, \theta)\, P(\mathbf{s} \mid \theta)$$

---

## Naïve Approach (Intractable)

Expanding Step 4 explicitly:

$$P(\mathbf{x} \mid \theta) = \sum_{\mathbf{s}} \pi_{s_1} b_{s_1}(x_1) \prod_{t=1}^{T-1} p_{s_t s_{t+1}} b_{s_{t+1}}(x_{t+1})$$

This requires summing over **all possible state sequences**: $s_1 \in \{1,\dots,n\}$, $s_2 \in \{1,\dots,n\}$, …, $s_T \in \{1,\dots,n\}$ — giving $n^T$ combinations in total.

> [!warning] Intractability
> The complexity is $\mathcal{O}(n^T \cdot T)$. For $n = 5$ states and $T = 100$ observations, this means $5^{100} \approx 10^{69}$ terms — completely intractable. The **Forward Algorithm** solves this in $\mathcal{O}(n^2 T)$ using dynamic programming.

---

## Efficient Solution: Forward Algorithm

Thanks to the [[Directed Acyclic Graph (DAG)]] structure (trellis), we can use dynamic programming.

**Define forward variable:**
$$\alpha_i(t) = P(x_1 \dots x_t, s_t = i \mid \theta)$$

**Recurrence → [[HMM - Forward Procedure]]:**

Initialization: $\alpha_j(1) = b_j(x_1) \pi_j$

Induction: $\alpha_j(t+1) = \left(\sum_{i=1}^n \alpha_i(t) p_{ij}\right) b_j(x_{t+1})$

**Likelihood:**
$$P(\mathbf{x} \mid \theta) = \sum_{j=1}^n \alpha_j(T)$$

---

## Backward Algorithm

**Define backward variable:**
$$\beta_i(t) = P(x_{t+1} \dots x_T \mid s_t = i, \theta)$$

**Recurrence → [[HMM - Backward Procedure]]:**

Initialization: $\beta_i(T) = 1$

Induction: $\beta_i(t) = \sum_{j=1}^n p_{ij} b_j(x_{t+1}) \beta_j(t+1)$

**Alternative likelihood computations:**
$$P(\mathbf{x} \mid \theta) = \sum_{i=1}^n \pi_i \beta_i(1) = \sum_{i=1}^n \alpha_i(t)\beta_i(t) \quad \forall t$$

---

## Important Quantity: $P(\mathbf{x}, s_t = i \mid \theta) = \alpha_i(t)\beta_i(t)$

**Derivation:**

$$\begin{align}
P(\mathbf{x}, s_t = i \mid \theta)
&= P(x_1 \dots x_t,\ s_t = i,\ x_{t+1} \dots x_T \mid \theta) \\[6pt]
&= P(x_{t+1} \dots x_T \mid x_1 \dots x_t,\ s_t = i,\ \theta)\ P(x_1 \dots x_t,\ s_t = i \mid \theta) \\[6pt]
&= P(x_{t+1} \dots x_T \mid s_t = i,\ \theta)\ P(x_1 \dots x_t,\ s_t = i \mid \theta) \\[6pt]
&= \beta_i(t)\ \alpha_i(t)
\end{align}$$

- Line 1: expand $\mathbf{x} = [x_1 \dots x_t, x_{t+1} \dots x_T]$ and include $s_t = i$
- Line 2: chain rule of probability
- Line 3: Markov property — future observations depend only on the current state $s_t = i$, not on the past observations $x_1 \dots x_t$
- Line 4: recognise $\beta_i(t) = P(x_{t+1}\dots x_T \mid s_t = i, \theta)$ and $\alpha_i(t) = P(x_1 \dots x_t, s_t = i \mid \theta)$

This quantity is central to [[HMM - Parameter Estimation (Baum-Welch)]] — it allows computing the posterior probability of being in state $i$ at time $t$:

$$\gamma_i(t) = P(s_t = i \mid \mathbf{x}, \theta) = \frac{\alpha_i(t)\,\beta_i(t)}{P(\mathbf{x} \mid \theta)} = \frac{\alpha_i(t)\,\beta_i(t)}{\sum_{j=1}^n \alpha_j(t)\,\beta_j(t)}$$

---

## Application: Isolated Word Recognition

Compute $P(\mathbf{x} \mid \theta^{(w)})$ for each reference word $w$ → select the word with **highest likelihood**.

---

*Part of → [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
