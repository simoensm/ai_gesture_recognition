---
tags: [recommender-systems, collaborative-filtering, KNN, matrix-factorization]
---
# Collaborative Filtering

**Collaborative filtering (CF)** recommends items based on the collective behaviour of users — it finds patterns in the ratings matrix without using any item content. The core assumption: users who agreed in the past tend to agree in the future.

**See also:** [[Concepts/Recommender_Systems]] | [[Concepts/Matrix_Factorization]] | [[Course - Sequence Modeling/Application - ItemRank (Collaborative Filtering)]]

---

## Two Branches

### Memory-Based CF

Stores the full ratings matrix and computes similarities at query time.

**User-based** — find users similar to the active user, borrow their ratings:
$$\hat{r}_{ui} = \mu_u + \frac{\sum_{v \in \mathcal{N}(u)} \text{sim}(u,v)\,(r_{vi} - \mu_v)}{\sum_{v \in \mathcal{N}(u)} |\text{sim}(u,v)|}$$

**Item-based** — find items similar to the target item (Sarwar et al. 2001 → [[Papers/Sarwar2001_ItemCF]]):
$$\hat{r}_{ui} = \mu_i + \frac{\sum_{j \in \mathcal{N}(i)} \text{sim}(i,j)\,(r_{uj} - \mu_j)}{\sum_{j \in \mathcal{N}(i)} |\text{sim}(i,j)|}$$

**Similarity metrics:** Pearson correlation, cosine, adjusted cosine, MSD, Jaccard, Spearman — see [[Workflow/12_Rating_Bias_and_Statistics]]

### Model-Based CF

Learns a compact model from the ratings matrix:
- **Matrix Factorization** (SVD, NMF) → [[Concepts/Matrix_Factorization]]
- **Rule-based / Bayesian classifiers**
- **Neural collaborative filtering**

---

## Key Weaknesses

| Problem | Description | Mitigation |
|---|---|---|
| **Cold start** | New users/items have no ratings | [[Concepts/Content_Based_Filtering]] or [[Papers/Schein2002_ColdStart]] |
| **Sparsity** | Most ratings are missing; similarity estimates noisy | Matrix factorization, regularisation |
| **Scalability** | All-pair similarity is $\mathcal{O}(n^2)$ | ItemBased (items < users), MF |
| **Popularity bias** | Popular items dominate recommendations | Novelty re-ranking, ILD post-filter |

---

## Project Results (MLSMM2156)

| Method | CV RMSE |
|---|---|
| UserBased Pearson k=80 | 0.8280 |
| UserBased Spearman k=80 | 0.8335 |
| ItemBased MSD k=40 | 0.8254 |

CF was outperformed by content-based (0.7344) and hybrid (0.7308) approaches on this dataset, primarily due to 97% sparsity.

---

## References

[[Papers/Resnick1994_GroupLens]] | [[Papers/Sarwar2001_ItemCF]] | [[Papers/Herlocker2004_EvaluatingCF]] | [[Papers/Fkih2021_SimilarityMeasures]]
