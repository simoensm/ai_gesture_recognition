---
title: "Statistical Learning Theory"
tags: [foundations, learning-theory, vc-dimension, pac-learning, generalisation, bias-variance]
created: 2026-05-22
---

# Statistical Learning Theory

> [!abstract] The question it answers
> When does a learning algorithm *generalise*? How many samples do you need? What's the fundamental tradeoff between model complexity and accuracy?

**Key references:** Vapnik (1995) *The Nature of Statistical Learning Theory* — [[Papers/Vapnik1995_SLT|Vapnik1995]]; Valiant (1984) PAC learning

---

## 1. The Learning Problem

**Setup:**
- Data drawn i.i.d.: $(x_i, y_i) \sim P(x, y)$, unknown distribution
- Hypothesis class $\mathcal{H}$: the family of functions the learner can choose from
- **Training error (empirical risk):** $\hat{R}(h) = \frac{1}{n}\sum_{i=1}^n \ell(h(x_i), y_i)$
- **True error (expected risk):** $R(h) = \mathbb{E}_{(x,y)\sim P}[\ell(h(x), y)]$

**Goal:** Find $h \in \mathcal{H}$ with small $R(h)$, using only the training data.

**Generalisation gap:** $R(h) - \hat{R}(h)$ — how much the model degrades on unseen data.

---

## 2. VC Dimension

The **Vapnik-Chervonenkis (VC) dimension** of $\mathcal{H}$ is the size of the largest set of points that $\mathcal{H}$ can **shatter** (correctly classify in all $2^n$ possible labellings).

| Model | VC Dimension |
|---|---|
| Threshold classifiers on $\mathbb{R}$ | 1 |
| Linear classifiers in $\mathbb{R}^d$ | $d + 1$ |
| $k$-NN | $\infty$ |
| Decision stumps | 2 |
| Neural networks | $\mathcal{O}(W \log W)$ where $W$ = parameters |

**Fundamental Theorem:**
$$R(h) \leq \hat{R}(h) + \mathcal{O}\left(\sqrt{\frac{\text{VC}(\mathcal{H})\log n + \log(1/\delta)}{n}}\right)$$

with probability $\geq 1 - \delta$.

→ Higher VC dimension = more expressive = needs more data to generalise.

---

## 3. PAC Learning (Probably Approximately Correct)

Introduced by Valiant (1984). A concept class $\mathcal{C}$ is **PAC learnable** if for any $\varepsilon, \delta > 0$, there exists an algorithm that, given $n \geq \text{poly}(1/\varepsilon, 1/\delta, d, \text{size})$ samples, outputs $h$ with $P(R(h) > \varepsilon) \leq \delta$.

**Key result:** A finite hypothesis class is PAC learnable with $n \geq \frac{1}{\varepsilon}\ln\frac{|\mathcal{H}|}{\delta}$ samples.

**Agnostic PAC:** When the target function is not in $\mathcal{H}$ — more realistic.

---

## 4. Bias-Variance Decomposition

For squared loss and a fixed test point $x$:
$$\mathbb{E}[(y - \hat{f}(x))^2] = \underbrace{\text{Bias}[\hat{f}(x)]^2}_{\text{underfitting}} + \underbrace{\text{Var}[\hat{f}(x)]}_{\text{overfitting}} + \underbrace{\sigma^2}_{\text{irreducible noise}}$$

| Term | Definition | Cause |
|---|---|---|
| **Bias** | $(\mathbb{E}[\hat{f}(x)] - f(x))^2$ | Model too simple, wrong assumptions |
| **Variance** | $\mathbb{E}[(\hat{f}(x) - \mathbb{E}[\hat{f}(x)])^2]$ | Model too complex, sensitive to training data |
| **Noise** | $\sigma^2 = \mathbb{E}[(y - f(x))^2]$ | Irreducible, inherent in data |

**Classical tradeoff:** Increasing model complexity ↓ Bias, ↑ Variance.

> [!note] Modern observation
> Over-parameterised models (neural networks with millions of parameters) can generalise well despite near-zero training error — the "double descent" phenomenon. Classical bias-variance analysis applies in the under-parameterised regime.

---

## 5. Regularisation & Structural Risk Minimisation

Instead of minimising training error alone, add a **complexity penalty**:
$$\hat{h} = \arg\min_{h \in \mathcal{H}} \left[\hat{R}(h) + \lambda \cdot \Omega(h)\right]$$

This is **Structural Risk Minimisation (SRM)** (Vapnik): choose the hypothesis class complexity to balance empirical risk and generalisation bound.

- $\lambda \uparrow$ → smoother model, higher bias, lower variance
- $\lambda \downarrow$ → more complex model, lower bias, higher variance

---

## Connections

- Regularisation → [[Concepts/Bias_Variance_Regularisation|Overfitting & Regularisation]]
- MAP ↔ regularisation → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- Cross-validation to estimate $R(h)$ → [[Concepts/Cross_Validation|Cross-Validation]]
- SVM maximises margin → directly from VC theory → [[Concepts/Kernel_Methods_SVM|SVM]]
- [[Papers/Vapnik1995_SLT|Vapnik 1995]]
