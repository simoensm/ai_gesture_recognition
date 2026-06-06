---
title: "Factorization Meets the Neighborhood: A Multifaceted Collaborative Filtering Model"
authors: ["Koren, Yehuda"]
year: 2008
journal: "KDD 2008"
citekey: Koren2008
tags: [literature, matrix-factorization, SVDpp, implicit-feedback, collaborative-filtering, netflix-prize]
status: read
doi: "10.1145/1401890.1401944"
---

# Factorization Meets the Neighborhood: A Multifaceted Collaborative Filtering Model

> [!info] Metadata
> **Authors:** Yehuda Koren | **Year:** 2008 | **Venue:** KDD, pp. 426–434

---

## Abstract

Presents **SVD++** — an extension of standard matrix factorization that integrates implicit feedback (the set of items a user has rated, regardless of rating values). Introduces the **asymmetric SVD** variant and demonstrates that combining latent factor and neighbourhood models outperforms either alone.

---

## SVD++ Model

$$\hat{r}_{ui} = \mu + b_u + b_i + \mathbf{q}_i^\top\!\left(\mathbf{p}_u + |\mathcal{I}_u|^{-1/2}\sum_{j\in\mathcal{I}_u}\mathbf{y}_j\right)$$

- $\mathcal{I}_u$ = set of items user $u$ has rated (implicit feedback)
- $\mathbf{y}_j$ = implicit item factor vector
- The sum $|\mathcal{I}_u|^{-1/2}\sum_j \mathbf{y}_j$ encodes **which items** the user has interacted with, orthogonal to **how** they rated them

---

## Key Finding

SVD++ consistently outperforms standard SVD on the Netflix Prize. The implicit feedback term adds information that the explicit ratings don't contain (e.g., browsing and skipping behaviour).

---

## Relevance to Project

`SVDppModel` in the project implements this. Results: CV RMSE 0.7890 (vs SVD 0.7913) — better mean but higher variance fold-to-fold. The tuned SVD actually ranks first in Friedman rank averaging despite being slightly worse in mean RMSE.

---

## Connections

[[Concepts/Matrix_Factorization]] | [[Papers/Koren2009_MatrixFactorization]] | [[Papers/Lee1999_NMF]] | [[Codes/05_Models_Latent_Factor]]
