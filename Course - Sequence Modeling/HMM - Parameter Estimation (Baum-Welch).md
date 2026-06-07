---
tags:
  - HMM
  - EM
  - Baum-Welch
  - parameter-estimation
  - sequence-modeling
---

Tags: #HMM #EM #Baum-Welch #parameter-estimation #learning

---

## Problem

Given an observation sequence $\mathbf{x}$, find model parameters $\theta = \{\mathbf{\Pi}, \mathbf{P}, \mathbf{B}\}$ that **maximize the likelihood** $P(\mathbf{x} \mid \theta)$.

No closed-form solution exists → use the **Baum-Welch algorithm** (a special case of the EM algorithm).

![[Screenshot 2026-06-01 at 15.15.52.png|503]]

---

## Key Quantities

**Probability of visiting state $i$ at time $t$:**
$$\gamma_i(t) = P(s_t = i \mid \mathbf{x}, \theta) = \frac{\alpha_i(t)\beta_i(t)}{\sum_j \alpha_j(t)\beta_j(t)}$$

**Probability of traversing arc $(i,j)$ at time $t$:**
$$\gamma_{ij}(t) = P(s_t = i,\ s_{t+1} = j \mid \mathbf{x}, \theta)$$

**Derivation of $\gamma_{ij}(t)$:**

$$\begin{align}
\gamma_{ij}(t) &= P(s_t = i,\ s_{t+1} = j \mid \mathbf{x}, \theta) \\[6pt]
&= \frac{P(x_1 \dots x_t,\ s_t = i,\ s_{t+1} = j,\ x_{t+1} \dots x_T \mid \theta)}{P(\mathbf{x} \mid \theta)} \\[6pt]
&= \frac{P(s_{t+1} = j,\ x_{t+1} \dots x_T \mid x_1 \dots x_t,\ s_t = i,\ \theta)\ P(x_1 \dots x_t,\ s_t = i \mid \theta)}{P(\mathbf{x} \mid \theta)} \\[6pt]
&= \frac{P(s_{t+1} = j,\ x_{t+1} \dots x_T \mid s_t = i,\ \theta)\ \alpha_i(t)}{P(\mathbf{x} \mid \theta)} \\[6pt]
&= \frac{P(x_{t+1} \dots x_T \mid s_t = i,\ s_{t+1} = j,\ \theta)\ P(s_{t+1} = j \mid s_t = i,\ \theta)\ \alpha_i(t)}{P(\mathbf{x} \mid \theta)} \\[6pt]
&= \frac{P(x_{t+1} \dots x_T \mid s_{t+1} = j,\ \theta)\ p_{ij}\ \alpha_i(t)}{P(\mathbf{x} \mid \theta)} \\[6pt]
&= \frac{P(x_{t+1} \mid x_{t+2} \dots x_T,\ s_{t+1} = j,\ \theta)\ P(x_{t+2} \dots x_T \mid s_{t+1} = j,\ \theta)\ p_{ij}\ \alpha_i(t)}{P(\mathbf{x} \mid \theta)} \\[6pt]
&= \frac{\alpha_i(t)\ b_j(x_{t+1})\ p_{ij}\ \beta_j(t+1)}{\displaystyle\sum_{k=1}^{n} \alpha_k(t)\,\beta_k(t)}
\end{align}$$

- Line 2: Bayes' rule — divide joint by $P(\mathbf{x}\mid\theta)$
- Line 3: chain rule
- Line 4: Markov property — past $x_1\dots x_t$ is irrelevant given $s_t=i$; recognise $\alpha_i(t)$
- Line 5: chain rule on the numerator
- Line 6: Markov property — future given $s_{t+1}=j$ does not depend on $s_t=i$; substitute $p_{ij}$
- Line 7: chain rule to split $x_{t+1}$ from $x_{t+2}\dots x_T$
- Line 8: recognise $b_j(x_{t+1})$ and $\beta_j(t+1)$; expand $P(\mathbf{x}\mid\theta) = \sum_k \alpha_k(t)\beta_k(t)$

---

## Re-Estimation Formulas

**Initial state probabilities:**
$$\hat{\pi}_i = \gamma_i(1)$$ (the probability of starting from i)

**Transition probabilities:**

$$\hat{p}_{ij} = \frac{\text{expected number of transitions from } i \text{ to } j}{\text{expected number of transitions out of state } i} = \frac{\displaystyle\sum_{t=1}^{T-1} P(s_t = i,\ s_{t+1} = j \mid \mathbf{x}, \hat{\theta})}{\displaystyle\sum_{t=1}^{T-1} P(s_t = i \mid \mathbf{x}, \hat{\theta})} = \frac{\displaystyle\sum_{t=1}^{T-1} \gamma_{ij}(t)}{\displaystyle\sum_{t=1}^{T-1} \gamma_i(t)}$$

**Emission probabilities:**

$$\hat{b}_i(o_k) = \frac{\displaystyle\sum_{t=1}^{T} P(s_t = i,\ x_t = o_k \mid \mathbf{x}, \hat{\theta})}{\displaystyle\sum_{t=1}^{T} P(s_t = i \mid \mathbf{x}, \hat{\theta})} = \frac{\displaystyle\sum_{t=1}^{T} P(s_t = i \mid \mathbf{x}, \hat{\theta})\,\delta(x_t = o_k)}{\displaystyle\sum_{t=1}^{T} P(s_t = i \mid \mathbf{x}, \hat{\theta})} = \frac{\displaystyle\sum_{\{t\,:\,x_t = o_k\}} \gamma_i(t)}{\displaystyle\sum_{t=1}^{T} \gamma_i(t)}$$

where $\delta(x_t = o_k) = 1$ if $x_t = o_k$ and $0$ otherwise — it selects only the time steps where observation $o_k$ was emitted, collapsing the full sum to $\sum_{\{t\,:\,x_t=o_k\}}$.

---

## Algorithm (EM Structure)

Repeat until convergence:
1. **E-step**: Compute $\alpha_i(t)$, $\beta_i(t)$, $\gamma_i(t)$, $\gamma_{ij}(t)$ using [[HMM - Forward Procedure]] and [[HMM - Backward Procedure]]
2. **M-step**: Re-estimate $\hat{\pi}_i$, $\hat{p}_{ij}$, $\hat{b}_i(o_k)$ using the formulas above

---

## Properties

- **Likelihood increases** at each iteration (guaranteed by EM theory)
- Converges to a **local maximum** (not necessarily global)
- Sensitive to initialization

---

*Part of → [[Hidden Markov Model (HMM)]] | [[HMM - Forward Procedure]] | [[HMM - Backward Procedure]] | [[MOC - Modelling Sequential Data]]*
