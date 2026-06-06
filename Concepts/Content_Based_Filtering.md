---
tags: [recommender-systems, content-based, NLP, profiling, TF-IDF]
---
# Content-Based Filtering

**Content-based filtering** recommends items similar to those a user has previously liked, based on **item attributes** and the **target user's own ratings** — no other users' data is needed.

**See also:** [[Concepts/Recommender_Systems]] | [[Concepts/TF-IDF]] | [[Workflow/10_Content_Based_Theory]]

---

## Three-Component Architecture

```
Information Source (raw item descriptions)
        ↓
┌──────────────────┐
│  CONTENT ANALYZER│  → numerical item feature vectors
└──────────────────┘
        ↓
┌──────────────────┐
│  PROFILE LEARNER │  ← user ratings
└──────────────────┘
        ↓
┌──────────────────────┐
│  FILTERING COMPONENT │  → ranked recommendations
└──────────────────────┘
```

---

## Content Analyzer — Feature Extraction

Item attributes fall into four types:

| Type | Example | Encoding |
|---|---|---|
| Nominal | Genres | One-hot encoding |
| Ordinal | IMDB rank | Ordinal / normalised |
| Interval | Year, length | Min-max normalisation |
| Textual | Tags, description | TF-IDF → [[Concepts/TF-IDF]], LDA |

**Text mining pipeline:** Raw text → Tokenisation → Stop-word removal → Stemming → Frequency filtering → Vectorisation (TF-IDF or LDA)

**Tag genome** (MovieLens-specific): 1,128 continuous genome tag relevance scores per movie — the richest content representation, significantly outperforming genre one-hot. See [[Papers/Vig2012_TagGenome]].

---

## Profile Learner

The user profile is a **trained model** mapping item features to predicted ratings:

| Rating type | ML task | Approach |
|---|---|---|
| Continuous | Regression | Ridge, Linear Regression, SVR |
| Binary | Classification | Logistic Regression, SVM, Naïve Bayes |

**Weighted average profile vector** (explainable):

$$\vec{p}_u = \frac{\sum_{i \in \text{rated}} r_i \cdot \vec{v}_i}{\sum_{i \in \text{rated}} r_i}$$

**Ridge regressor** (used in the project):

$$\mathbf{w}_u^* = \arg\min_{\mathbf{w}} \sum_{i \in \mathcal{I}_u} (r_{ui} - \mathbf{w}^\top \mathbf{x}_i)^2 + \alpha|\mathbf{w}|^2$$

Preferred over OLS (unstable with 1,128 features), Lasso (spurious sparsity on correlated genome tags), and RidgeCV (slower, same result).

---

## Cold-Start Advantage

Content-based is the only approach that works for **new or unpopular items** — no co-ratings needed. This is its primary advantage over CF:

$$\mathbf{R} = \begin{pmatrix} \cdots & \mathbf{?} & \cdots \end{pmatrix} \quad \text{(empty column = new item)}$$

CF sees nothing; content-based uses item attributes directly. See [[Papers/Schein2002_ColdStart]].

---

## Pros and Cons

**Advantages:** handles new/unpopular items, explainable ("because you like Sci-Fi"), no privacy leak between users, any sklearn regressor works.

**Disadvantages:** cold-start for new users (no profile without ratings), over-specialisation (no serendipity), domain knowledge required for feature engineering.

---

## Project Results (MLSMM2156)

| Model | CV RMSE | Feature set |
|---|---|---|
| Genre one-hot | 0.8955 | 20 binary cols |
| Genome SVD (n=100) | 0.7438 | 100 latent dims |
| Genome full, Ridge α=20 | 0.7353 | 1,128 dims |
| **Genome full + stats, Ridge α=20** | **0.7344** | 1,132 dims |

The genome tag representation completely dominates genre-based features — a statistically significant difference (Friedman p=0.00128). See [[Workflow/10_Content_Based_Theory]] | [[Webapp/03_Cascade_Architecture]].

---

## References

[[Papers/Pazzani2007_ContentBased]] | [[Papers/Vig2012_TagGenome]] | [[Papers/Deldjoo2016_VisualFeatures]] | [[Papers/Salton1988_TFIDF]]
