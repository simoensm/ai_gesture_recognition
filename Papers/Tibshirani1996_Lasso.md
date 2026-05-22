---
title: "Regression Shrinkage and Selection via the Lasso"
authors: ["Tibshirani, Robert"]
year: 1996
citekey: "Tibshirani1996_Lasso"
doi: "10.1111/j.2517-6161.1996.tb02080.x"
venue: "Journal of the Royal Statistical Society, Series B, 58(1), 267–288"
type: article
tags: [literature, lasso, l1-regularisation, sparsity, feature-selection, regression, shrinkage, foundational]
status: read
relevance: high
created: 2026-05-22
---

# Regression Shrinkage and Selection via the Lasso

> [!info] Metadata
> **Authors:** Robert Tibshirani (Stanford)
> **Year:** 1996
> **Venue:** Journal of the Royal Statistical Society, Series B, 58(1), 267–288

---

## Abstract

Introduces the **Lasso** (Least Absolute Shrinkage and Selection Operator) — a regularised regression method that imposes an L1 penalty on the coefficients. Unlike ridge regression (L2), the L1 penalty produces **sparse solutions** with many coefficients exactly equal to zero, simultaneously performing regression and feature selection.

---

## Key Contributions

### The Lasso Objective
$$\hat{\beta}^{lasso} = \arg\min_\beta \frac{1}{2}\|y - X\beta\|_2^2 \quad \text{subject to} \quad \|\beta\|_1 \leq t$$

Equivalent Lagrangian form:
$$\hat{\beta}^{lasso} = \arg\min_\beta \frac{1}{2n}\|y - X\beta\|_2^2 + \lambda\|\beta\|_1$$

The constraint $\|\beta\|_1 \leq t$ defines an L1 ball (a diamond in 2D); the solution is often at a **corner** where some $\beta_j = 0$ exactly.

### Geometric Insight

```
β₂
│  ●  = unconstrained OLS solution
│  □  = L1 ball (lasso constraint)
│  ○  = L2 ball (ridge constraint)
│
│    ● (OLS)
│   /
│  □── ← lasso hits corner → β₁ = 0 (sparse!)
│  ─────────────────── β₁
```

Because the L1 ball has **corners on the axes**, the optimal solution tends to land exactly on a corner — making one or more coefficients exactly zero. The L2 ball is round — no corners — so ridge rarely produces exact zeros.

### Soft-Thresholding
The coordinate-wise update for the lasso (coordinate descent):
$$\hat\beta_j \leftarrow \text{sign}(\hat\rho_j)(|\hat\rho_j| - \lambda)_+$$
where $\hat\rho_j = \frac{1}{n}x_j^\top(y - X_{-j}\hat\beta_{-j})$ is the partial residual correlation and $(z)_+ = \max(z, 0)$.

---

## Regularisation Path

As $\lambda$ decreases from $\lambda_{max}$ to 0:
- $\lambda_{max}$: all $\beta_j = 0$ (maximum sparsity)
- $\lambda = 0$: recovers OLS (all features included)
- Intermediate $\lambda$: some features selected, others zeroed

The **LARS algorithm** (Efron et al., 2004) computes the entire lasso path efficiently in one pass, with the same cost as a single OLS fit.

---

## Comparison: Lasso vs. Ridge vs. Best Subset

| Method | Penalty | Sparsity | Complexity | Stability |
|---|---|---|---|---|
| **Best subset** | $\|\beta\|_0 \leq k$ | Exact | $\mathcal{O}(2^d)$ (NP-hard) | Unstable (discrete) |
| **Ridge** | $\|\beta\|_2^2$ | No (shrinks but non-zero) | Closed form | Stable |
| **Lasso** | $\|\beta\|_1$ | Yes (some exact zeros) | Convex (LARS/coord. descent) | Moderate |
| **Elastic Net** | $\alpha\|\beta\|_1 + (1-\alpha)\|\beta\|_2^2$ | Yes | Convex | Good for correlated features |

---

## Probabilistic Interpretation

The lasso is equivalent to **MAP estimation with a Laplace prior**:
$$p(\beta_j) = \frac{\lambda}{2}\exp(-\lambda|\beta_j|) \quad \text{(Laplace / double exponential)}$$

The Laplace distribution has a sharp peak at zero → sparsity-inducing prior. Compare to ridge = MAP with Gaussian prior $p(\beta_j) \propto \exp(-\lambda\beta_j^2/2)$.

---

## Extensions

- **Group Lasso** (Yuan & Lin, 2006): Selects groups of features together — useful when features form natural groups
- **Adaptive Lasso:** Use data-driven weights $|\beta_j|^{-\gamma}$ per feature — oracle property (correct variable selection asymptotically)
- **SCAD / MCP:** Non-convex penalties that reduce bias for large coefficients
- **Bayesian Lasso:** Full Bayesian treatment with Laplace prior via auxiliary variables (Gibbs sampling)

---

## Connections

- MAP interpretation → [[Foundations/Maximum_Likelihood_Estimation|MLE & MAP]]
- Covered in depth → [[Papers/Hastie2009_ESL|ESL Ch. 3 & 18]]
- Ridge regression → [[Concepts/Linear_Regression|Linear Regression]]
- Regularisation → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Same author (Tibshirani) → [[Papers/Hastie2009_ESL|ESL]]
