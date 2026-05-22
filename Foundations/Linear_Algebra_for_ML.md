---
title: "Linear Algebra for Machine Learning"
tags: [foundations, linear-algebra, matrices, eigenvalues, svd, pca]
created: 2026-05-22
---

# Linear Algebra for Machine Learning

> [!abstract] The language of ML
> Data is matrices. Models transform vectors. Training is geometry. PCA, SVD, covariance, attention — all linear algebra.

---

## 1. Vectors & Norms

A data point $x \in \mathbb{R}^d$. Key norms:

| Norm | Formula | Geometry | ML use |
|---|---|---|---|
| L1 $\|x\|_1$ | $\sum_i |x_i|$ | Manhattan | Lasso regularisation, sparse |
| L2 $\|x\|_2$ | $\sqrt{\sum_i x_i^2}$ | Euclidean | Ridge, DTW cost, distances |
| L∞ $\|x\|_\infty$ | $\max_i |x_i|$ | Chebyshev | Robustness, adversarial |
| Frobenius $\|A\|_F$ | $\sqrt{\sum_{i,j} A_{ij}^2}$ | Matrix L2 | Weight decay |

---

## 2. Matrix Operations Critical for ML

**Matrix multiplication** $C = AB$: $C_{ij} = \sum_k A_{ik} B_{kj}$

**Transpose:** $(AB)^\top = B^\top A^\top$

**Inverse:** $A^{-1}A = I$ — exists iff $A$ is square and full rank

**Trace:** $\text{tr}(A) = \sum_i A_{ii}$ — sum of eigenvalues

**Determinant:** $\det(A)$ — signed volume of the linear transformation

---

## 3. Eigendecomposition

For a square matrix $A$: $A v = \lambda v$ where $\lambda$ is an eigenvalue and $v$ is the corresponding eigenvector.

For symmetric positive semidefinite (PSD) $A = A^\top \succeq 0$:
$$A = Q \Lambda Q^\top, \quad Q^\top Q = I, \quad \Lambda = \text{diag}(\lambda_1, \ldots, \lambda_d)$$

**Covariance matrix** $\Sigma = \frac{1}{n}X^\top X$ is always PSD. Its eigenvectors are the **principal components** (PCA directions). Its eigenvalues are the explained variances.

---

## 4. Singular Value Decomposition (SVD)

For any $m \times n$ matrix $A$:
$$A = U \Sigma V^\top$$

- $U \in \mathbb{R}^{m \times m}$: left singular vectors (orthonormal)
- $\Sigma \in \mathbb{R}^{m \times n}$: diagonal, singular values $\sigma_1 \geq \sigma_2 \geq \ldots \geq 0$
- $V \in \mathbb{R}^{n \times n}$: right singular vectors (orthonormal)

**Truncated SVD** (rank-$k$ approximation):
$$A \approx U_k \Sigma_k V_k^\top = \sum_{i=1}^k \sigma_i u_i v_i^\top$$

This is the **best rank-$k$ approximation** (Eckart-Young theorem).

**ML uses:** PCA, LSA (text), collaborative filtering, Kabsch algorithm (→ [[Concepts/Kabsch_Algorithm|Kabsch]]), pseudo-inverse, attention matrix decomposition.

---

## 5. PCA via SVD

Given centred data matrix $X \in \mathbb{R}^{n \times d}$ (zero-mean columns):

1. Compute SVD: $X = U\Sigma V^\top$
2. Principal components = columns of $V$ (right singular vectors)
3. Projected data = $XV = U\Sigma$
4. Variance explained by $k$-th PC = $\sigma_k^2 / \sum_i \sigma_i^2$

PCA finds the directions of **maximum variance** in the data.

**Connection to eigendecomposition:**
$$\text{Cov}(X) = \frac{1}{n}X^\top X = \frac{V\Sigma^2 V^\top}{n}$$

The eigenvectors of the covariance matrix = the right singular vectors of $X$.

---

## 6. Quadratic Forms & PSD Matrices

$x^\top A x$ is a quadratic form. If $A \succeq 0$ (PSD):
- $x^\top A x \geq 0$ for all $x$
- All eigenvalues $\geq 0$
- Can be Cholesky-factored: $A = LL^\top$

**Mahalanobis distance:**
$$d_M(x, \mu) = \sqrt{(x-\mu)^\top \Sigma^{-1}(x-\mu)}$$

Measures distance relative to the covariance structure. Used in multivariate Gaussian, anomaly detection.

---

## 7. Gradients & Jacobians

For ML we often need:
- $\nabla_x (a^\top x) = a$
- $\nabla_x (x^\top A x) = (A + A^\top)x = 2Ax$ if $A$ symmetric
- $\nabla_W (\|Wx - b\|^2) = 2W^\top(Wx - b)$

**Jacobian** $J \in \mathbb{R}^{m \times n}$: $J_{ij} = \partial f_i / \partial x_j$ — the multi-output generalisation of the gradient. Used in backpropagation.

---

## Connections

- SVD → [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]] ($3 recognizer)
- SVD → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]] (PCA, t-SNE)
- Eigendecomposition → [[Concepts/Attention_Transformer|Attention]] (QKV are linear projections)
- Covariance matrix → [[Foundations/Probability_Theory|Probability Theory]] (multivariate Gaussian)
- Gradients → [[Optimisation_Theory]] (backpropagation)
