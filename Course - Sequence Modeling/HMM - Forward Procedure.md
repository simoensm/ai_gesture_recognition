

Tags: #HMM #forward #dynamic-programming #alpha

---

## Definition

The **forward variable** $\alpha_i(t)$ is the joint probability of:
- Observing $x_1, \dots, x_t$ up to time $t$
- Being in state $i$ at time $t$

$$\alpha_i(t) = P(x_1 \dots x_t, s_t = i \mid \theta)$$

---

## Recurrence

**Initialization** ($t=1$):
$$\alpha_j(1) = b_j(x_1) \cdot \pi_j$$

**Induction** ($t = 1, \dots, T-1$):
$$\alpha_j(t+1) = \left(\sum_{i=1}^n \alpha_i(t) \cdot p_{ij}\right) \cdot b_j(x_{t+1})$$

**Interpretation:** $\alpha_j(t+1)$ = probability of all paths ending in state $j$ at $t+1$ and generating $x_1 \dots x_{t+1}$.

---

## Derivation

$$\alpha_j(t+1) = P(x_1 \dots x_{t+1}, s_{t+1} = j)$$

$$= \sum_i P(s_{t+1}=j, x_{t+1} \mid s_t=i) \cdot \alpha_i(t)$$

$$= \sum_i P(x_{t+1} \mid s_{t+1}=j) \cdot P(s_{t+1}=j \mid s_t=i) \cdot \alpha_i(t)$$

$$= b_j(x_{t+1}) \cdot \sum_i p_{ij} \cdot \alpha_i(t)$$

---

## Complexity

$O(n^2 T)$ — polynomial in both $n$ (states) and $T$ (length), compared to $O(n^T)$ for the naïve approach.

---

## Relation to Other Procedures

- Combined with [[HMM - Backward Procedure]]: $P(\mathbf{x}, s_t = i) = \alpha_i(t)\beta_i(t)$
- Used in [[HMM - Parameter Estimation (Baum-Welch)]] for re-estimation

---

*Part of → [[HMM - Likelihood Computation]] | [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
