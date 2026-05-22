# HMM - Parameter Estimation (Baum-Welch)

Tags: #HMM #EM #Baum-Welch #parameter-estimation #learning

---

## Problem

Given an observation sequence $\mathbf{x}$, find model parameters $\theta = \{\mathbf{\Pi}, \mathbf{P}, \mathbf{B}\}$ that **maximize the likelihood** $P(\mathbf{x} \mid \theta)$.

No closed-form solution exists → use the **Baum-Welch algorithm** (a special case of the EM algorithm).

---

## Key Quantities

**Probability of visiting state $i$ at time $t$:**
$$\gamma_i(t) = P(s_t = i \mid \mathbf{x}, \theta) = \frac{\alpha_i(t)\beta_i(t)}{\sum_j \alpha_j(t)\beta_j(t)}$$

**Probability of transitioning from $i$ to $j$ at time $t$:**
$$\gamma_{ij}(t) = P(s_t=i, s_{t+1}=j \mid \mathbf{x}, \theta) = \frac{\alpha_i(t) \cdot p_{ij} \cdot b_j(x_{t+1}) \cdot \beta_j(t+1)}{\sum_k \alpha_k(t)\beta_k(t)}$$

---

## Re-Estimation Formulas

**Initial state probabilities:**
$$\hat{\pi}_i = \gamma_i(1)$$

**Transition probabilities:**
$$\hat{p}_{ij} = \frac{\sum_{t=1}^{T-1} \gamma_{ij}(t)}{\sum_{t=1}^{T-1} \gamma_i(t)}$$

> = expected transitions $i \to j$ / expected transitions out of $i$

**Emission probabilities:**
$$\hat{b}_i(o_k) = \frac{\sum_{\{t : x_t = o_k\}} \gamma_i(t)}{\sum_{t=1}^T \gamma_i(t)}$$

> = expected emissions of $o_k$ in state $i$ / total expected emissions in state $i$

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
