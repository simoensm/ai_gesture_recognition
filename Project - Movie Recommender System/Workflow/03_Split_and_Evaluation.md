---
tags:
  - recommender-systems
  - workflow
  - evaluation
  - cross-validation
  - MLSMM2156
---
# Train/Test Split & Evaluation Protocol

How we measure model performance rigorously, without data leakage.

---

## Why Not Just a Single Train/Test Split?

A single 75/25 split gives one RMSE number. But that number depends on *which* 25% was held out by luck. With a different random seed, the RMSE could differ by ±0.003 or more.

**K-Fold Cross-Validation** solves this: we split the data into K groups ("folds"), train K times (each time holding out a different fold as the test set), and report **mean ± std RMSE across all K folds**. This gives a more reliable estimate of true generalisation performance.

---

## K-Fold Cross-Validation (Main Protocol)

```
Full dataset (180,000 ratings)
│
├─── Fold 1 test (20%)  │  Train on folds 2+3+4+5  →  RMSE₁
├─── Fold 2 test (20%)  │  Train on folds 1+3+4+5  →  RMSE₂
├─── Fold 3 test (20%)  │  Train on folds 1+2+4+5  →  RMSE₃
├─── Fold 4 test (20%)  │  Train on folds 1+2+3+5  →  RMSE₄
└─── Fold 5 test (20%)  │  Train on folds 1+2+3+4  →  RMSE₅

Reported: mean(RMSE₁…₅) ± std(RMSE₁…₅)
```

**Implementation:** Surprise's `KFold` with `random_state=42, shuffle=True`. Defined in `ablation_study.py`.

| Model family | n_folds |
|---|---|
| All models (default) | **5** |
| UserBased, ItemBased | **3** (O(n²) similarity is too slow for 5 folds) |

> Using 3 folds for CF models means their confidence intervals are slightly wider (±0.0011 vs ±0.0006 for 5-fold models). This is acknowledged in the [[07_Statistical_Assessment]].

---

## Random vs. Temporal Splitting

We use **random** rating-level splitting (not temporal/chronological).

**Why random?**
- Our goal is **rating prediction accuracy**, not "predict the user's next watch"
- Random splitting better reflects general recommendation quality
- Temporal splitting would be preferred if the task were sequential next-item prediction

(Meng et al. 2020 showed that splitting strategy can change the relative ranking of models — this choice matters.)

---

## Beyond-Accuracy Metrics

RMSE alone doesn't tell the full story. A model that recommends the same 10 blockbusters to every user has low RMSE but is useless. Three additional metrics are computed in `ablation_study.py`:

### Intra-List Diversity (ILD)
Measures how **genre-diverse** a top-N recommendation list is.

```
ILD(u) = (2 / N(N-1)) × Σ_{i<j} (1 − cosine_similarity(genre_i, genre_j))
```

- 0 = all items in the list have identical genres (no diversity)
- 1 = all items are from completely different genres (maximum diversity)
- Computed on a **top-10 list** for each user

### Novelty
Measures how **obscure** the recommended items are (opposite of popularity bias).

```
Novelty = E[ -log₂ p(i) ]   where p(i) = #ratings(i) / n_users
```

Higher = more niche items recommended. Popular films like Pulp Fiction have low self-information; obscure films have high self-information.

### Coverage
Fraction of the full item catalogue that appears in **at least one** recommendation list across all users.

```
Coverage = |unique recommended items| / |total items|
```

A model with low coverage shows the same films to every user (popularity bias).

---

## Leave-One-Out (LOO) — Hit Rate

A ranking-based protocol (separate from RMSE):

1. For each user, **withhold one rating** from the training set
2. The model must rank the withheld item in its **top-N list** (N=40)
3. Hit Rate = fraction of users where the withheld item appears in the top-40

This tests whether the model *retrieves* the right item, not just whether it predicts the right score. The content-based models score best here (Hybrid: 0.064, i.e., 6.4% of users' held-out item appears in top-40).

---

## Anti-Testset for Diversity Metrics

ILD, Novelty, and Coverage are computed on the **anti-testset**: for each user, score all *unrated* items (items not in their training set). Then take the top-10. This simulates what a real user would receive as recommendations.

Computing this for all ~1,000 users × ~8,700 items is slow for content-based models (they need a forward pass per user per item). So we sample:
- **300 users** for CF and latent-factor models
- **100 users** for content-based models (slower per-call)
- **600 candidate items per user** for content-based models

---

> [!WARNING]+ ❓ Assessor Question — What is data leakage and how did you prevent it?
> Data leakage occurs when information from the test set "leaks" into training, making the model appear better than it really is. Two main risks in this project:
> 1. **Item statistics:** Computing item quality stats (mean rating, count) from the *full* dataset would include test-set ratings. The `ContentBasedWithStats` class computes these statistics exclusively from `trainset.ir` (training fold ratings only), re-computing them at each CV fold.
> 2. **Feature computation from full data:** The `_item_stats_scaled()` helper in `models.py` uses the full ratings file and explicitly warns it must NOT be used inside CV. Only `ContentBasedWithStats.fit()` (which uses the trainset) is used in CV.

> [!WARNING]+ ❓ Assessor Question — Why is the Surprise library used for splitting?
> Surprise's `KFold` splits at the **rating level** (each (user, item, rating) triple is independently assigned to a fold). This means a user can appear in both train and test sets — which is correct for rating prediction (we evaluate on some held-out ratings from users we've seen). It is *not* a user-level split (where entire users are held out), because we evaluate generalisation over items, not over new users.

> [!WARNING]+ ❓ Assessor Question — What does RMSE = 0.73 actually mean?
> On average, our predicted rating is about 0.73 stars away from the true rating on the 0.5–5.0 scale. To put it in context: the global-mean baseline (always predicting 3.5) gets RMSE ≈ 1.05. Random guessing gets RMSE ≈ 1.4. Our best model's RMSE of 0.73 means the model is substantially personalised — it learns user preferences well enough to cut the error nearly in half compared to always predicting the average.

---

**See also:** [[Workflow/06_Ablation_Study]] | [[Workflow/07_Statistical_Assessment]] | [[Codes/08_Ablation_and_Assessment_Code]]
