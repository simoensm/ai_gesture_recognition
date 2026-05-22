# HMM - Decoding (Viterbi Algorithm)

Tags: #HMM #viterbi #dynamic-programming #decoding

---

## Problem

Given an observation sequence $\mathbf{x}$ and model $\theta$, find the **most likely hidden state sequence**:

$$\mathbf{s}^* = \arg\max_\mathbf{s} P(\mathbf{s} \mid \mathbf{x}, \theta) = \arg\max_\mathbf{s} P(\mathbf{s}, \mathbf{x} \mid \theta)$$

This is called **decoding** or speech segmentation.

---

## Sub-Problem: Best State at Each Time t

Define:
$$\gamma_i(t) = P(s_t = i \mid \mathbf{x}, \theta) = \frac{\alpha_i(t)\beta_i(t)}{\sum_j \alpha_j(t)\beta_j(t)}$$

Best state at time $t$: $s^*(t) = \arg\max_i \gamma_i(t)$

⚠️ This gives the **marginally** best state at each $t$, but the sequence $\{s^*(t)\}$ may not be a valid state sequence (transitions may be impossible).

---

## Viterbi: Best Overall Sequence

**Define Viterbi variable:**
$$\delta_j(t) = \max_{s_1 \dots s_{t-1}} P(s_1 \dots s_{t-1}, x_1 \dots x_{t-1}, s_t = j, x_t \mid \theta)$$

The maximum joint probability of generating $x_1 \dots x_t$ and being in state $j$ at $t$.

---

## Recurrence

**Initialization** ($t=1$):
$$\delta_j(1) = \pi_j b_j(x_1)$$

**Induction** ($t = 1, \dots, T-1$):
$$\delta_j(t+1) = \max_i \{ \delta_i(t) \cdot p_{ij} \cdot b_j(x_{t+1}) \}$$

**Backtracking pointer:**
$$\psi_j(t+1) = \arg\max_i \{ \delta_i(t) \cdot p_{ij} \cdot b_j(x_{t+1}) \}$$

---

## Path Recovery (Backtracking)

$$s^*(T) = \arg\max_i \delta_i(T)$$

$$s^*(t-1) = \psi_{s^*(t)}(t)$$

Work backward from $T$ to $1$.

---

## Log-Space Variant

To avoid numerical underflow, compute in **log space**:
- Products become sums
- $\log \delta$ replaces $\delta$

---

## Connection to [[Dynamic Programming]]

Viterbi IS dynamic programming on the HMM trellis ([[Directed Acyclic Graph (DAG)]]).
- States = nodes
- Time steps = levels
- $\delta$ = optimal cost function (here: max probability)

Compare to [[Bellman Recurrence Relations]] (min-cost), Viterbi uses **max-probability**.

---

*Part of → [[Hidden Markov Model (HMM)]] | [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
