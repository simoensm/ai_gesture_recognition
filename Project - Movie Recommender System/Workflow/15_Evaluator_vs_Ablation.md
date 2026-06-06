---
tags: [recommender-systems, evaluation, LOO, ablation, performance]
---
# Evaluator vs Ablation Study — Key Differences

**Related:** [[Workflow/06_Ablation_Study]] | [[Workflow/07_Statistical_Assessment]] | [[Codes/08_Ablation_and_Assessment_Code]]

---

## Summary Table

| | Evaluator (`02_evaluator.ipynb`) | Ablation Study (`ablation_study.py`) |
|---|---|---|
| Accuracy | Single 75/25 split | 5-fold CV |
| Ranking eval | Full LOO (~8.7M predictions) | Sampled anti-testset (~180K for CB) |
| Users scored | All ~1,000 | 300 (100 for CB) |
| Items per user | All unrated (~8,744) | 600 cap for CB, full for CF |
| Passes | 3 | 1 per fold |

---

## Why LOO Is Expensive

In `02_evaluator.ipynb`, `generate_loo_top_n()` uses `LeaveOneOut`, which for every user withholds one rating and scores **all unrated items** to build the top-N list.

For ~1,000 users × ~8,744 movies = **≈8.7 million predictions**, just to compute hit rate.

The ablation uses `_partial_anti_testset()` which samples:
- `N_DIV_USERS = 300` users (not all 1,000)
- `CAND_ITEMS_CB = 600` items per user for content-based models

So the ablation's equivalent step is at most **300 × 600 = 180,000 predictions** for CB — a **~50× reduction**.

---

## Why Content-Based Models Are Slow

Each `ContentBased.estimate()` call does a `pandas.loc` lookup and a dot product per item. At 8.7M calls in the LOO path this is brutal; at 180K in the ablation path it's manageable.

From `configs.py`:
> _"content-based models are slow per prediction (pandas.loc per call)"_

---

## Evaluator's Three Passes

1. **Split RMSE/MAE** — one fit + 25% test
2. **LOO hit rate** — one fit + 8.7M anti-testset predictions
3. **Full novelty** — another pass over anti-testset

The ablation runs **one 5-fold CV loop** that produces RMSE, MAE, ILD, coverage, and novelty all in a single sampled anti-testset pass per fold.

---

## Implication for Hit Rate Comparison

If you want the ablation to be as rigorous as the evaluator on ranking quality, you'd need to accept the full anti-testset cost — or accept that hit rate is only sampled in the ablation.

**`UserBased` vs `KNNWithMeans`:** `UserBased` is a slower Python reimplementation of `KNNWithMeans` whose only purpose is to unlock Spearman and Jaccard. For cosine/Pearson/MSD, always prefer `KNNWithMeans`, which is why `EvalConfig` uses `KNNWithMeans` for those and only uses `UserBased` for Spearman/Jaccard configurations.
