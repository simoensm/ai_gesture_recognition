---
title: "Mathematics for Machine Learning"
authors: ["Deisenroth, Marc Peter", "Faisal, A. Aldo", "Ong, Cheng Soon"]
year: 2020
citekey: "Deisenroth2020_MML"
doi: "10.1017/9781108679930"
venue: "Cambridge University Press"
type: book
tags: [literature, mathematics, linear-algebra, probability, optimisation, machine-learning, textbook, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Mathematics for Machine Learning

> [!info] Metadata
> **Authors:** Marc Peter Deisenroth, A. Aldo Faisal, Cheng Soon Ong
> **Year:** 2020
> **Publisher:** Cambridge University Press
> **Free PDF:** [mml-book.github.io](https://mml-book.github.io)

---

## Abstract

A comprehensive, self-contained textbook covering the mathematical prerequisites for modern machine learning. Deliberately bridges the gap between undergraduate mathematics and the ML research literature. Covers linear algebra, analytic geometry, matrix decompositions, vector calculus, probability, distributions, and continuous optimisation — then applies each to regression, PCA, Gaussian mixture models, and SVMs.

---

## Structure

### Part I — Mathematical Foundations

| Chapter | Topic | Key concepts |
|---|---|---|
| 2 | **Linear Algebra** | Vector spaces, linear maps, basis, rank, inner products |
| 3 | **Analytic Geometry** | Norms, distances, angles, projections, orthogonal complements |
| 4 | **Matrix Decompositions** | Determinant, eigenvalues/vectors, Cholesky, eigendecomposition, SVD |
| 5 | **Vector Calculus** | Gradients, Jacobians, Hessians, chain rule, backpropagation |
| 6 | **Probability & Distributions** | Random variables, Gaussian, conjugate priors, exponential family |
| 7 | **Continuous Optimisation** | Unconstrained, convex, gradient descent, stochastic, Lagrangians |

### Part II — Central Machine Learning Problems

| Chapter | Application | Foundations used |
|---|---|---|
| 8 | **Linear Regression** | MLE, MAP, Bayesian linear regression, projections |
| 9 | **PCA** | SVD, eigendecomposition, variance maximisation |
| 10 | **Gaussian Mixture Models** | EM algorithm, Bayesian inference |
| 11 | **SVMs** | Lagrange multipliers, kernel trick, duality |

---

## Key Equations Worth Knowing

### SVD
$$A = U\Sigma V^\top \quad U \in \mathbb{R}^{m\times m},\; \Sigma \in \mathbb{R}^{m\times n},\; V \in \mathbb{R}^{n\times n}$$
Every matrix has an SVD. Columns of $U$ are left singular vectors (eigenvectors of $AA^\top$); columns of $V$ are right singular vectors (eigenvectors of $A^\top A$).

### Gradient of Matrix Operations
- $\nabla_A \text{tr}(AB) = B^\top$
- $\nabla_A \log\det(A) = A^{-\top}$
- Chain rule: $\frac{d}{dx}(f \circ g)(x) = \frac{\partial f}{\partial g}\frac{\partial g}{\partial x}$ (scalar) → Jacobians for vector case

### Lagrangian Duality (SVM)
$\mathcal{L}(w,b,\alpha) = \frac{1}{2}\|w\|^2 - \sum_i \alpha_i [y_i(w^\top x_i + b) - 1]$, $\alpha_i \geq 0$

---

## Why This Book

Unlike most ML textbooks, MML *derives* everything from first principles. Reading chapters 4 (matrix decompositions) and 7 (optimisation) before studying any algorithm pays dividends throughout the course.

---

## Connections

- Linear algebra details → [[Foundations/Linear_Algebra_for_ML|Linear Algebra for ML]]
- Probability coverage → [[Foundations/Probability_Theory|Probability Theory]]
- Optimisation → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- MLE and Bayesian regression → [[Foundations/Maximum_Likelihood_Estimation|MLE]]
- PCA chapter → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]
- SVM chapter → [[Concepts/Kernel_Methods_SVM|Kernel Methods & SVM]]
- EM chapter → [[Concepts/Clustering_EM|Clustering & EM Algorithm]]
