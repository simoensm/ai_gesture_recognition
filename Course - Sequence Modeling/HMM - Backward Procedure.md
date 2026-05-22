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

$$\beta_i(t) = P(x_{t+1} \dots x_T \mid s_t = i)$$

$$= \sum_j P(x_{t+1} \dots x_T, s_{t+1}=j \mid s_t=i)$$

$$= \sum_j P(x_{t+1} \mid s_{t+1}=j) \cdot P(s_{t+1}=j \mid s_t=i) \cdot P(x_{t+2} \dots x_T \mid s_{t+1}=j)$$

$$= \sum_j p_{ij} \cdot b_j(x_{t+1}) \cdot \beta_j(t+1)$$

---

## Usage

**Likelihood** (alternative computation):
$$P(\mathbf{x} \mid \theta) = \sum_i \pi_i \beta_i(1)$$

**Combined with forward variable:**
$$P(\mathbf{x}, s_t = i \mid \theta) = \alpha_i(t) \beta_i(t)$$

This is central to [[HMM - Parameter Estimation (Baum-Welch)]].

---

*Part of → [[HMM - Likelihood Computation]] | [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
