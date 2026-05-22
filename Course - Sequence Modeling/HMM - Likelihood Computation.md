# HMM - Likelihood Computation

Tags: #HMM #likelihood #forward #backward

---

## Problem

Given an observation sequence $\mathbf{x} = [x_1, \dots, x_T]^T$ and model $\theta = \{\mathbf{\Pi}, \mathbf{P}, \mathbf{B}\}$, compute:
$$P(\mathbf{x} \mid \theta)$$

---

## Naïve Approach (Intractable)

Sum over all possible hidden state sequences:

$$P(\mathbf{x} \mid \theta) = \sum_{\mathbf{s}} \pi_{s_1} b_{s_1}(x_1) \prod_{t=1}^{T-1} p_{s_t s_{t+1}} b_{s_{t+1}}(x_{t+1})$$

This requires summing over $n^T$ sequences → **exponential complexity**.

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

## Important Quantity

$$P(\mathbf{x}, s_t = i \mid \theta) = \alpha_i(t)\beta_i(t)$$

Used in [[HMM - Parameter Estimation (Baum-Welch)]].

---

## Application: Isolated Word Recognition

Compute $P(\mathbf{x} \mid \theta^{(w)})$ for each reference word $w$ → select the word with **highest likelihood**.

---

*Part of → [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
