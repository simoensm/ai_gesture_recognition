---
title: "Matrix Factorization Techniques for Recommender Systems"
authors: ["Koren, Yehuda", "Bell, Robert", "Volinsky, Chris"]
year: 2009
journal: "Computer (IEEE)"
citekey: Koren2009
tags: [literature, matrix-factorization, recommender-systems, SVD, collaborative-filtering, netflix-prize]
status: read
doi: ""
---

# Matrix Factorization Techniques for Recommender Systems

> [!info] Metadata
> **Authors:** Yehuda Koren, Robert Bell, Chris Volinsky
> **Year:** 2009 | **Venue:** IEEE Computer, Vol. 42(8), pp. 30–37
> **Citekey:** Koren2009

---

## Abstract

Presents matrix factorization as the dominant paradigm for collaborative filtering in recommender systems. Demonstrates that MF models are superior to classic nearest-neighbour techniques, offering flexibility to model additional information (implicit feedback, temporal dynamics, confidence levels). Describes the approach that won the Netflix Prize.

---

## Key Contributions

- Formal treatment of **SVD-based MF** for RecSys: $\hat{r}_{ui} = \mu + b_u + b_i + \mathbf{p}_u^\top \mathbf{q}_i$
- **Bias terms** $b_u$, $b_i$ capture systematic rating tendencies
- Extension to **implicit feedback** (which items a user has interacted with, regardless of rating)
- **Temporal dynamics** — user preferences and item popularity shift over time
- **Confidence weighting** for implicit feedback signals

## Training

SGD minimises regularised squared error:

$$\min \sum_{(u,i)\in\mathcal{K}} \left(r_{ui} - \mu - b_u - b_i - \mathbf{p}_u^\top\mathbf{q}_i\right)^2 + \lambda\left(\|\mathbf{p}_u\|^2 + \|\mathbf{q}_i\|^2 + b_u^2 + b_i^2\right)$$

---

## Relevance to Project

The foundational reference for all SVD and SVD++ models used in the MLSMM2156 project. The bias terms ($b_u$, $b_i$) are implemented in Surprise's `SVD` class and controlled by the `biased` hyperparameter.

---

## Connections

[[Concepts/Matrix_Factorization]] | [[Papers/Koren2008_FactorizationNeighborhood]] | [[Papers/Lee1999_NMF]] | [[Codes/05_Models_Latent_Factor]]
