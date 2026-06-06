# Project Overview

**Goal:** Build the most accurate MovieLens-based recommender system possible, measured by RMSE on held-out ratings, while avoiding overfitting and preserving diversity.

---

## The Problem

A recommender system predicts what rating a user would give to a movie they haven't seen yet. This is a **rating prediction** task:

- **Input:** A user's history of (movie, rating) pairs
- **Output:** A predicted rating for every unrated movie
- **Primary metric:** RMSE (Root Mean Squared Error) — lower is better

---

## Dataset at a Glance

See [[02_Data]] for full detail.

| Property | Value |
|---|---|
| Users | ~1,000 |
| Movies | ~8,744 |
| Ratings | ~180,000 |
| Rating scale | 0.5 to 5.0 (10 half-star levels) |
| Sparsity | > 97% (most user/movie pairs have no rating) |
| Global mean | ~3.5 |

---

## Model Families Explored

| Family | Core Idea | Best CV RMSE |
|---|---|---|
| **Baseline** | Always return a constant | ~1.0+ |
| **User-based CF** | Find similar users, aggregate their ratings | 0.8280 |
| **Item-based CF** | Find similar items to what you already rated | 0.8254 |
| **Latent Factor (SVD)** | Decompose the rating matrix into user/item vectors | 0.7913 |
| **Latent Factor (SVD++)** | SVD + implicit "which items you rated" signal | 0.7890 |
| **Content-Based** | Per-user regression on item features (genome tags) | 0.7346 |
| **Hybrid** | Linear blend of SVD + Content | **0.7308** |

The **Hybrid model wins**. The gain comes from combining two orthogonal signals: genome tags tell us *what kind* of movie it is; SVD tells us *how users collectively rate* it.

---

## Key Takeaways

1. **Genome features dominate content-based models.** 1,128 crowd-sourced perceptual tags beat synopsis, director, cast, and poster features combined.
2. **Hyperparameter tuning is a first-order effect.** Fixing SVD's regularisation from default (λ=0.02) to optimal (λ=0.05) saved more RMSE than switching model families.
3. **The optimal hybrid is 80% content / 20% SVD.** Content already explains ~80% of predictable variance; SVD adds a small orthogonal correction.
4. **No model dominates all dimensions.** CF models are more diverse (ILD); content models are more accurate but create filter bubbles.

---

> [!WARNING]+ ❓ Assessor Question — Why use RMSE as the primary metric?
> **RMSE** penalises large errors more heavily than small ones (because errors are squared before averaging). In a rating prediction context, a prediction of 4.5 on a true 1.0 is far more damaging than a prediction of 3.6 on a true 3.0. This asymmetry is desirable: we care more about not grossly misjudging a user's taste than about marginal errors near the true value. RMSE also has a clean mathematical relationship to the MSE loss minimised by regression models, making it the standard benchmark metric for MovieLens tasks.

> [!WARNING]+ ❓ Assessor Question — What is the rating scale and why does it matter?
> Ratings go from 0.5 to 5.0 in 0.5 steps (10 possible values). The scale is **discrete and bounded**, which means: (1) predictions must be clipped to [0.5, 5.0] before evaluation; (2) the distribution is right-skewed (most ratings are 3–4, few are 0.5–1); (3) per-user mean-centering in CF removes individual baseline differences (a user who always rates 4–5 vs. one who rates 2–3 can have the same relative preferences).

---

**Next:** [[02_Data]] | [[Workflow/03_Split_and_Evaluation]] | [[Workflow/08_Results_and_Decision_Making]]
