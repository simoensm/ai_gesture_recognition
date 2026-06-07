---
tags:
  - recommender-systems
  - workflow
  - results
  - RMSE
  - MLSMM2156
---
# Results & Decision Making

Complete RMSE results per model family and the reasoning behind each design choice.

---

## Best Model Per Family

| Family | Best Config | CV RMSE | Notes |
|---|---|---|---|
| **Hybrid** | SVD α=0.2 + Genome full Ridge α=20 | **0.7308 ± 0.0015** | Overall winner |
| Content-Based | Genome full + stats, Ridge α=20 | 0.7346 ± 0.0015 | Best standalone |
| Latent Factor (SVD++) | f=150, λ=0.05, η=0.01 | 0.7890 ± 0.0024 | Best pure CF |
| Latent Factor (SVD) | f=100, λ=0.05, η=0.01 | 0.7913 ± 0.0015 | |
| Item-based CF | MSD, k=40 | 0.8254 ± 0.0011 | Best neighbourhood |
| User-based CF | Pearson (KNNWithMeans), k=80 | 0.8280 ± 0.0023 | |
| Latent Factor (NMF) | f=10 | 0.8229 ± 0.0021 | |
| Content (TMDB) | TMDB full, Ridge | 0.8878 ± 0.0015 | |

---

## Model Family Comparison

### Collaborative Filtering (CF) — 0.82–0.84
CF models find similar users/items and aggregate ratings. They work purely from the rating matrix — no content information.

**Why CF performs worse than content models:**
1. **97% sparsity** — most user pairs share very few co-rated movies, making similarity estimates unreliable
2. **Cold start** — new items with no ratings can't be recommended
3. **Scale** — O(|U|²) or O(|I|²) complexity

### Latent Factor Models (SVD/SVD++) — 0.79
SVD decomposes the rating matrix into user and item embeddings. It's smarter than KNN because it captures *latent preference dimensions* rather than raw similarity.

**Key insight:** Default Surprise settings (λ=0.02) are sub-optimal. Tuning to λ=0.05 saves 0.017 RMSE — more than switching from SVD to SVD++.

### Content-Based — 0.73
Per-user Ridge on genome tags. Bypasses sparsity entirely by using item features.

**Key insight:** Genome full (1,128 raw tags) beats genome SVD-compressed. Ridge already handles high dimensions via L2 regularisation — pre-compressing loses information.

### Hybrid — 0.7308 ✓
20% SVD + 80% Content. The SVD arm adds a small but consistent orthogonal correction:
- Content captures "what kind of movie" (genome tag profile)
- SVD captures "how users collectively rate" (residual collaborative signal)

---

## Decision Tree: Why Did We Choose Each Component?

```
START: What's the primary signal?
│
├── Rating matrix only → Collaborative Filtering
│   ├── Need speed / scalability → SVD (latent factors)
│   └── Need interpretability → KNN (neighbourhood)
│
├── Item features only → Content-Based
│   ├── Best features? → Genome full (1,128 perceptual tags)
│   └── Best regressor? → Ridge α=20 (L2-regularised, prevents overfitting)
│
└── Both → Hybrid
    └── Optimal mix? → 20% SVD + 80% Content (α=0.2)
```

---

## Why Genome Full Beats Everything Else

| Feature Type | Best RMSE | Why it fails |
|---|---|---|
| Genre one-hot (19 dims) | 0.8955 | Too coarse — Action/Sci-Fi doesn't predict taste |
| Release year | 0.93+ | Correlation with taste is indirect |
| Visual poster features | 0.9272 | Aesthetics ≠ preference; 47% missing data |
| Synopsis TF-IDF (500 dims) | 0.9262 | Narrative words ≠ taste |
| TMDB full (671 dims) | 0.8878 | Factual metadata ≠ perceptual experience |
| **Genome full (1,128 dims)** | **0.7353** | Crowd-sourced *perceptual* attributes |

The genome is special because it was collected by asking MovieLens users *how much* each tag applies to each movie. The tags were chosen to represent the vocabulary of movie experience — exactly the vocabulary users use when forming preferences.

---

## Why α=0.2 is the Optimal Hybrid Mix

The content model already explains ~80% of predictable variance. SVD contributes a small **orthogonal** correction for:
- Films where collective ratings diverge from their content profile (production quality, cultural timing, events not captured by any tag)
- Users whose taste can be partially predicted from the collective behaviour of similar users

At α > 0.4, the weaker SVD predictions start diluting the content model's precision. At α < 0.1, the SVD signal is too diluted to add value.

---

## Feature Ablation Decision Points

| Decision | Choice | Reason |
|---|---|---|
| Genome full vs. SVD-compressed | **Full** | Ridge handles 1,128 dims natively; compression loses +0.007 RMSE |
| ContentBased vs. ContentBasedWithStats | **WithStats** | Item quality stats (Bayesian mean, count, std, high-ratio) add −0.007 RMSE |
| Ridge vs. RidgeCV | **Ridge α=20** | RidgeCV finds per-user alphas close to 20 → no benefit at extra cost |
| Ridge vs. Random Forest | **Ridge** | Ridge is faster and consistently better across all configurations |
| SVD vs. SVD++ in hybrid | **SVD** | SVD++ saves only 0.0002 RMSE in the hybrid context (negligible) |

---

> [!WARNING]+ ❓ Assessor Question — Why does the hybrid RMSE (0.7308) beat both standalone components (Content: 0.7346, SVD: 0.7913)?
> Because the two components capture **orthogonal variance**: content captures preference for content types (genome tags predict what the user would rate a film with given attributes); SVD captures residual collaborative patterns (how users in this community collectively deviate from content predictions). The two signals don't overlap, so combining them reduces variance on the errors that each individual model makes. This is the Jensen's inequality effect: the average of two imperfect predictors is better than either alone when their errors are uncorrelated.

> [!WARNING]+ ❓ Assessor Question — What are the remaining weaknesses of the hybrid model?
> 1. **Sparse users:** Users with < 10 ratings get unreliable Ridge solutions (underdetermined system). The model degrades toward an intercept-only prediction.
> 2. **Item cold start:** New movies with no ratings have zero SVD factors → SVD component predicts near the global mean. But they still have genome tags, so the content component still personalises.
> 3. **Filter bubble:** ILD = 0.663 — the hybrid produces genre-homogeneous lists. Users who love thrillers get a lot of thrillers. The 20% SVD weight partially mitigates this but doesn't solve it.
> 4. **Coverage:** 8% of the catalogue appears in any recommendation list. 92% of movies are never recommended to anyone.

> [!WARNING]+ ❓ Assessor Question — How does RidgeCV work and why didn't it help?
> RidgeCV fits a Ridge model for each candidate α value and selects the one that minimises cross-validation error *within* that user's training data. With the default grid {5, 10, 20, 25, 50, 100, 200}, most users' optimal α is found to be near 20 — the same value we use with fixed Ridge. So RidgeCV provides per-user tuning, but since users' optimal α values are clustered around 20, the average benefit is negligible while the runtime cost (fitting multiple models per user) is non-trivial.

---

**See also:** [[Workflow/09_Beyond_Accuracy]] | [[Workflow/05_Hyperparameter_Tuning]] | [[Workflow/07_Statistical_Assessment]]
