---
tags:
  - HMM
  - viterbi
  - dynamic-programming
  - decoding
  - sequence-modeling
---
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
Define the state sequence which maximizes the joint probability of generating the observations _x__t_ and observing the states _s__t_ up to time _t_–1, landing in state _j_, and emitting the observation _x__t_ at time _t_

---

## Recurrence

**Initialization** ($t=1$):
$$\delta_j(1) = \pi_j b_j(x_1)$$

**Induction** ($t = 1, \dots, T-1$) — derivation:

$\delta_j(t+1)$ is provided by the following forward recurrence relation:

$$\begin{align}
&\max_{s_1 \dots s_t} P(s_1 \dots s_{t+1} = j,\ x_1 \dots x_{t+1} \mid \theta) \\[6pt]
&= \max_{s_1 \dots s_t} P(s_{t+1} = j,\ x_{t+1} \mid s_1 \dots s_t,\ x_1 \dots x_t,\ \theta)\ P(s_1 \dots s_t,\ x_1 \dots x_t \mid \theta) \\[6pt]
&= \max_{s_1 \dots s_t} \left\{ P(x_{t+1} \mid s_{t+1} = j,\ \theta)\ P(s_{t+1} = j \mid s_t,\ \theta)\ P(s_1 \dots s_t,\ x_1 \dots x_t \mid \theta) \right\} \\[6pt]
&= \max_{s_t} \max_{s_1 \dots s_{t-1}} \left\{ b_{j}(x_{t+1})\ p_{s_t j}\ P(s_1 \dots s_t,\ x_1 \dots x_t \mid \theta) \right\} \\[6pt]
&= \max_{s_t} \left\{ b_j(x_{t+1})\ p_{s_t j} \max_{s_1 \dots s_{t-1}} P(s_1 \dots s_t,\ x_1 \dots x_t \mid \theta) \right\} \\[6pt]
&= \max_{s_t} \left\{ b_j(x_{t+1})\ p_{s_t j}\ \delta_{s_t}(t) \right\}
\end{align}$$

- Line 2: chain rule
- Line 3: Markov property + conditional independence of $x_{t+1}$ given $s_{t+1}$
- Line 4: substitute $b_j$ and $p_{s_t j}$; split the max over $s_t$ and $s_1\dots s_{t-1}$
- Line 5: $b_j(x_{t+1})$ does not depend on $s_1\dots s_{t-1}$, so pass it outside the inner max
- Line 6: recognise the inner max as $\delta_{s_t}(t)$

Which gives the recurrence:

$$\boxed{\delta_j(t+1) = \max_{i} \left\{ b_j(x_{t+1})\ p_{ij}\ \delta_i(t) \right\}}$$

**Backtracking pointer:**
$$\psi_j(t+1) = \arg\max_i \{ \delta_i(t) \cdot p_{ij} \cdot b_j(x_{t+1}) \}$$

---

## Path Recovery (Backtracking)

Once the forward pass is complete, we recover the most likely state sequence by **working backward** from $T$ to $1$:

$$s^*(T) = \arg\max_i \delta_i(T)$$

$$s^*(t) = \psi_{s^*(t+1)}(t+1) \quad \text{for } t = T-1, \dots, 1$$

Each pointer $\psi_j(t+1)$ records which previous state $i$ achieved the maximum in the induction step, so backtracking simply follows the chain of pointers from the best final state back to $t = 1$.

---

## Log-Space Variant

We can also apply dynamic programming to the **log-likelihood** instead of the likelihood directly. Taking the log of $\delta_j(t)$:

$$\log \delta_j(t+1) = \log b_j(x_{t+1}) + \max_i \left\{ \log p_{ij} + \log \delta_i(t) \right\}$$

The key advantages are:
- **Products become sums** — the recurrence is now additive, which is numerically much more stable
- **Avoids underflow** — multiplying many small probabilities over long sequences drives $\delta$ to zero in floating point; log-space prevents this
- The $\arg\max$ is unchanged, so backtracking works identically

---

## Connection to [[Dynamic Programming]]

Viterbi IS dynamic programming on the HMM trellis ([[Directed Acyclic Graph (DAG)]]).
- States = nodes
- Time steps = levels
- $\delta$ = optimal cost function (here: max probability)

Compare to [[Bellman Recurrence Relations]] (min-cost), Viterbi uses **max-probability**.

---

*Part of → [[Hidden Markov Model (HMM)]] | [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
