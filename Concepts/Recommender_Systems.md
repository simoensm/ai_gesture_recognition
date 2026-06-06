---
tags: [recommender-systems, machine-learning, personalization]
---
# Recommender Systems

A **recommender system** is a machine learning system that predicts the preference a user would give to an item, and uses those predictions to surface a personalised ranked list of items.

**Project:** [[Project - Movie Recommender System/00_HOME|RecSys MLSMM2156]]

---

## Taxonomy

```
Recommender System Approaches
├── Collaborative Filtering          → [[Concepts/Collaborative_Filtering]]
│   ├── Memory-based (KNN)
│   └── Model-based (MF, SVD, NMF)  → [[Concepts/Matrix_Factorization]]
├── Content-Based Filtering          → [[Concepts/Content_Based_Filtering]]
├── Hybrid                           → [[Papers/Burke2002_HybridRS]]
├── Demographic
├── Knowledge-based
└── Context-based
```

---

## Core Problem Formulation

Given a ratings matrix $\mathbf{R} \in \mathbb{R}^{m \times n}$ (users × items) with many missing entries (sparsity ≈ 95–99%), predict $r_{ui}$ for unobserved user-item pairs.

**Key challenges:**
- **Sparsity** — most users have rated only a tiny fraction of items
- **Cold start** — new users/items have no ratings → [[Papers/Schein2002_ColdStart]]
- **Scalability** — millions of users and items
- **Beyond accuracy** — diversity, novelty, coverage matter → [[Workflow/09_Beyond_Accuracy]]

---

## Evaluation Metrics

**Accuracy:** RMSE, MAE (lower is better)

$$\text{RMSE} = \sqrt{\frac{1}{|\mathcal{T}|}\sum_{(u,i)\in\mathcal{T}} (r_{ui} - \hat{r}_{ui})^2}$$

**Ranking:** Hit Rate @ N (Leave-One-Out), Precision@N, Recall@N

**Beyond accuracy:**
- **ILD** (Intra-List Diversity) — how diverse are the top-N recommendations?
- **Coverage** — what fraction of the catalogue is ever recommended?
- **Novelty** — how surprising are the recommendations?

See → [[Papers/Herlocker2004_EvaluatingCF]] | [[Papers/Castells2015_Novelty]]

---

## Dataset: MovieLens

Standard benchmark: [[Papers/Harper2016_MovieLens]]
- 1M ratings, ~6,000 users, ~4,000 movies
- Enriched with genome tags: [[Papers/Vig2012_TagGenome]]

---

## Related Concepts

[[Concepts/Collaborative_Filtering]] | [[Concepts/Matrix_Factorization]] | [[Concepts/Content_Based_Filtering]] | [[Concepts/TF-IDF]] | [[Course - Sequence Modeling/Application - ItemRank (Collaborative Filtering)]]
