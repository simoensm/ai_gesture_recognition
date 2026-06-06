---
tags: [recommender-systems, matrix-factorization, SVD, NMF, latent-factors, dimensionality-reduction]
---
# Matrix Factorization

**Matrix factorization (MF)** decomposes the ratings matrix $\mathbf{R} \approx \mathbf{U}\mathbf{V}^T$ into two low-rank factor matrices, discovering a compact **latent representation** of users and items.

**See also:** [[Concepts/Collaborative_Filtering]] | [[Concepts/Dimensionality_Reduction]] | [[Workflow/11_Latent_Factor_Theory]]

---

## Core Idea

$$\mathbf{R}_{m \times n} \approx \mathbf{U}_{m \times k}\,\mathbf{V}^T_{k \times n}$$

- $\mathbf{U}$: user factor matrix — each row $\mathbf{p}_u$ is a user's latent preference vector
- $\mathbf{V}$: item factor matrix — each row $\mathbf{q}_i$ is an item's latent feature vector
- $k \ll \min(m, n)$: number of latent factors (typically 20–200)

Predicted rating: $\hat{r}_{ui} = \mu + b_u + b_i + \mathbf{p}_u \cdot \mathbf{q}_i$

---

## Learning: Funk SVD (SGD)

Minimise regularised squared error:

$$\min_{\mathbf{U},\mathbf{V}} \sum_{(u,i)\in\mathcal{K}} (r_{ui} - \hat{r}_{ui})^2 + \lambda(\|\mathbf{p}_u\|^2 + \|\mathbf{q}_i\|^2 + b_u^2 + b_i^2)$$

Stochastic gradient descent update per observed rating:

$$\mathbf{p}_u \leftarrow \mathbf{p}_u + \eta\,(e_{ui}\,\mathbf{q}_i - \lambda\,\mathbf{p}_u)$$
$$\mathbf{q}_i \leftarrow \mathbf{q}_i + \eta\,(e_{ui}\,\mathbf{p}_u - \lambda\,\mathbf{q}_i)$$

where $e_{ui} = r_{ui} - \hat{r}_{ui}$.

Key hyperparameters: $k$ (n_factors), $\lambda$ (reg), $\eta$ (lr), n_epochs.

---

## SVD++ Extension

SVD++ adds implicit feedback (which items users have rated, regardless of the rating value):

$$\hat{r}_{ui} = \mu + b_u + b_i + \mathbf{q}_i^\top\!\left(\mathbf{p}_u + |\mathcal{I}_u|^{-1/2}\sum_{j\in\mathcal{I}_u}\mathbf{y}_j\right)$$

More powerful but harder to tune. Koren (2008) → [[Papers/Koren2008_FactorizationNeighborhood]]

---

## NMF (Non-Negative Matrix Factorization)

Constrains $\mathbf{U}, \mathbf{V} \geq 0$ — only additive combinations, no cancellation:
$$\mathbf{R} \approx \mathbf{U}\mathbf{V}^T, \quad U_{ij} \geq 0,\; V_{ij} \geq 0$$

Original paper: Lee & Seung (1999) → [[Papers/Lee1999_NMF]]

In RecSys: NMF has lower RMSE accuracy than SVD but produces more diverse recommendations (higher ILD), representing a different accuracy–diversity trade-off.

---

## Latent Factors: Interpretation

Latent factors are **not labelled** — they are discovered by the algorithm. They may loosely correspond to genre, mood, era, or style, but they are defined purely by the data structure.

> "Like Plato's cave: we observe ratings (shadows), but the latent factors are the unseen real causes."

See full theory → [[Workflow/11_Latent_Factor_Theory]]

---

## Project Results (MLSMM2156)

| Model | CV RMSE |
|---|---|
| NMF f=10 | 0.8229 |
| SVD default | 0.8148 |
| SVD tuned (λ=0.05, η=0.01, f=100) | 0.7913 |
| SVD++ tuned (f=150) | 0.7890 |

---

## References

[[Papers/Koren2009_MatrixFactorization]] | [[Papers/Koren2008_FactorizationNeighborhood]] | [[Papers/Lee1999_NMF]]
