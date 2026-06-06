---
title: "Learning the Parts of Objects by Non-Negative Matrix Factorization"
authors: ["Lee, Daniel D.", "Seung, H. Sebastian"]
year: 1999
journal: "NeurIPS 1999"
citekey: Lee1999
tags: [literature, NMF, matrix-factorization, dimensionality-reduction, recommender-systems]
status: read
---

# Learning the Parts of Objects by Non-Negative Matrix Factorization

> [!info] Metadata
> **Authors:** Lee, Seung | **Year:** 1999 | **Venue:** NeurIPS, Vol. 12, pp. 556–562

---

## Abstract

Introduces Non-Negative Matrix Factorization (NMF) as a method for learning parts-based representations. Demonstrates that non-negativity constraints lead to sparse, parts-based decompositions (e.g., face images → facial features) unlike PCA or ICA.

---

## NMF Formulation

$$\min_{\mathbf{W},\mathbf{H} \geq 0} \|\mathbf{V} - \mathbf{W}\mathbf{H}\|_F^2$$

Constraint: all entries of $\mathbf{W}$ and $\mathbf{H}$ must be $\geq 0$ — only additive combinations, no cancellation.

**Multiplicative update rules:**

$$H_{a\mu} \leftarrow H_{a\mu}\,\frac{(\mathbf{W}^\top \mathbf{V})_{a\mu}}{(\mathbf{W}^\top \mathbf{W} \mathbf{H})_{a\mu}}, \quad W_{ia} \leftarrow W_{ia}\,\frac{(\mathbf{V}\mathbf{H}^\top)_{ia}}{(\mathbf{W}\mathbf{H}\mathbf{H}^\top)_{ia}}$$

---

## In Recommender Systems

Applied to the ratings matrix: $\mathbf{R} \approx \mathbf{U}\mathbf{V}^T$ with $\mathbf{U}, \mathbf{V} \geq 0$.

Non-negativity means each item's latent vector is a **additive combination of "topics"** — more interpretable than SVD (which allows negative factors). However, NMF typically achieves higher RMSE than SVD.

**Project result:** NMF f=10 achieves CV RMSE 0.8229 but the **highest ILD (0.751)** of all models — trading accuracy for diversity.

---

## Connections

[[Concepts/Matrix_Factorization]] | [[Papers/Koren2009_MatrixFactorization]] | [[Codes/05_Models_Latent_Factor]] | [[Workflow/09_Beyond_Accuracy]]
