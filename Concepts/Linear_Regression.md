---
title: "Linear Regression"
tags: [concept, supervised-learning, linear-regression, ols, ridge, lasso, regularisation, regression]
created: 2026-05-22
---

# Linear Regression

> The simplest supervised learning model — predict a continuous output as a weighted sum of inputs. Despite its simplicity, it is the foundation for understanding generalisation, regularisation, and probabilistic modelling.

---

## 1. Model

$$\hat{y} = w^\top x + b = \sum_{j=1}^d w_j x_j + b$$

In matrix form (absorbing bias into $w$ by appending a 1 to $x$):
$$\hat{y} = Xw, \quad X \in \mathbb{R}^{n \times d}, \; w \in \mathbb{R}^d, \; y \in \mathbb{R}^n$$

**Hypothesis class:** all affine functions $f(x) = w^\top x + b$.

---

## 2. Ordinary Least Squares (OLS)

Minimise the sum of squared residuals:
$$\hat{w}_{OLS} = \arg\min_w \|y - Xw\|_2^2 = \arg\min_w \sum_{i=1}^n (y_i - w^\top x_i)^2$$

**Closed-form solution (Normal Equations):**
$$\hat{w} = (X^\top X)^{-1} X^\top y$$

Derived by setting $\nabla_w \|y - Xw\|^2 = -2X^\top(y - Xw) = 0$.

**Pseudoinverse:** When $X^\top X$ is singular (underdetermined or collinear features), use SVD: $\hat{w} = V\Sigma^+ U^\top y$ — minimum-norm solution.

### Geometric Interpretation
$\hat{y} = Xw$ is the **orthogonal projection** of $y$ onto the column space of $X$. The residual $y - \hat{y}$ is perpendicular to all columns of $X$.

---

## 3. Probabilistic Interpretation (MLE)

Assume Gaussian noise: $y_i = w^\top x_i + \varepsilon_i$, $\varepsilon_i \sim \mathcal{N}(0, \sigma^2)$

The likelihood:
$$p(y | X, w) = \prod_i \mathcal{N}(y_i; w^\top x_i, \sigma^2)$$

MLE = minimising negative log-likelihood = **minimising MSE**. So OLS is maximum likelihood under Gaussian noise.

→ See [[Foundations/Maximum_Likelihood_Estimation|MLE & MAP]]

---

## 4. Regularisation

### Ridge Regression (L2)
$$\hat{w}_{Ridge} = \arg\min_w \|y - Xw\|^2 + \lambda\|w\|_2^2$$

Closed form: $\hat{w} = (X^\top X + \lambda I)^{-1} X^\top y$

- Shrinks all coefficients towards zero
- Always invertible even when $X^\top X$ is singular
- **Probabilistic view:** MAP with Gaussian prior $w \sim \mathcal{N}(0, \sigma^2/\lambda \cdot I)$

### Lasso Regression (L1)
$$\hat{w}_{Lasso} = \arg\min_w \|y - Xw\|^2 + \lambda\|w\|_1$$

- **Produces sparse solutions** — some $w_j = 0$ exactly → automatic feature selection
- No closed form; solved via coordinate descent or LARS algorithm
- **Probabilistic view:** MAP with Laplace prior $w_j \sim \text{Laplace}(0, 1/\lambda)$
- Paper → [[Papers/Tibshirani1996_Lasso|Tibshirani (1996)]]

### Elastic Net
$$\hat{w} = \arg\min_w \|y - Xw\|^2 + \lambda_1\|w\|_1 + \lambda_2\|w\|_2^2$$
Combines L1 sparsity with L2 stability — useful when features are correlated.

### Comparison

| Method | Regularisation | Solution | Sparsity | Suitable when |
|---|---|---|---|---|
| OLS | None | Closed form | No | $n \gg d$, no collinearity |
| Ridge | L2 | Closed form | No | Collinear features |
| Lasso | L1 | Iterative | Yes | Feature selection needed |
| Elastic Net | L1 + L2 | Iterative | Yes | Correlated + sparse |

---

## 5. Polynomial Regression

Feature map $\phi(x) = [1, x, x^2, \ldots, x^p]^\top$. Then apply linear regression to $\phi(x)$:
$$\hat{y} = w^\top \phi(x) = \sum_{k=0}^p w_k x^k$$

**Linear in parameters, non-linear in input.** This is the key insight that extends to kernel methods — any feature map $\phi$ gives a richer hypothesis class while keeping the linear machinery.

---

## 6. Model Complexity and Generalisation

Increasing polynomial degree $p$:
- Low $p$: high bias (underfits, misses true pattern)
- High $p$: low bias but high variance (overfits training set, fails on new data)

The validation curve shows test error rising as $p$ grows past the optimal — classic bias-variance trade-off.
→ See [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]

---

## 7. Assumptions and Diagnostics

**Gauss-Markov theorem:** OLS is the Best Linear Unbiased Estimator (BLUE) under:
1. Linearity in parameters
2. Strict exogeneity: $\mathbb{E}[\varepsilon | X] = 0$
3. Homoscedasticity: $\text{Var}[\varepsilon_i] = \sigma^2$ (constant variance)
4. No multicollinearity: $X$ has full column rank

**Diagnostics:**
- **Residual plot:** Residuals vs. fitted values — should be random scatter (no pattern = linearity + homoscedasticity)
- **Q-Q plot:** Normal quantile plot of residuals — checks Gaussian assumption
- **VIF (Variance Inflation Factor):** Detects multicollinearity; VIF > 10 indicates a problem

---

## Connections

- Probabilistic view → [[Foundations/Maximum_Likelihood_Estimation|MLE & MAP]]
- Lasso paper → [[Papers/Tibshirani1996_Lasso|Tibshirani 1996]]
- Extends to classification → [[Concepts/Logistic_Regression|Logistic Regression]]
- Regularisation details → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- SVD for pseudoinverse → [[Foundations/Linear_Algebra_for_ML|Linear Algebra for ML]]
- Bayesian linear regression → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- Reference → [[Papers/Hastie2009_ESL|ESL Ch. 3]], [[Papers/Bishop2006_PRML|PRML Ch. 3]]
