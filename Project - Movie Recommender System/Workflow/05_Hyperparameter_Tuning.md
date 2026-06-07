---
tags:
  - recommender-systems
  - workflow
  - hyperparameter-tuning
  - MLSMM2156
---
# Hyperparameter Tuning Strategy

How hyperparameters were chosen and why specific ranges were explored.

---

## Core Principle

> **All hyperparameter selection uses only the training folds. The test fold is never used to pick parameters.**

This prevents overfitting to the evaluation set. Every reported RMSE is the average over held-out folds that were never seen during parameter selection.

---

## Search Strategy: Grid Search over CV Folds

We use **grid search** (exhaustive enumeration over a candidate list) rather than Bayesian optimisation. Reasons:
- Grid search is transparent and fully reproducible
- With < 10 hyperparameters per model, the grid is small enough to be exhaustive
- Each grid point runs K-fold CV → we get mean ± std, not a point estimate

The grid for each family was designed based on **theoretical priors** (expected regularisation scale, known sweet spots from literature).

---

## Per-Model Tuning Grids

### User-Based CF

| Hyperparameter | Grid | Best | Why this range? |
|---|---|---|---|
| `k` (neighbours) | {20, 40, 80} | **80** (Pearson) | Too few: high variance; too many: noise from distant users |
| `sim` (similarity) | {MSD, Pearson, Spearman, Cosine, Jaccard} | **Pearson** | Mean-centering removes scale bias |
| `min_support` | {1, 3} | 3 (minimal impact) | Require at least 3 co-rated items for a valid similarity |

Best: **Pearson k=80, CV RMSE = 0.8280**

### Item-Based CF

| Hyperparameter | Grid | Best | Why? |
|---|---|---|---|
| `k` | {10, 20, 40} | **40** (MSD) | More neighbours = more stable for item-based |
| `similarity_method` | {adj_cosine, pearson, msd} | **MSD** | Item vectors rated by different users → MSD more stable than Pearson |

Best: **MSD k=40, CV RMSE = 0.8254**

### Content-Based Ridge (α sweep)

α controls the L2 penalty strength. Larger α = stronger regularisation = smaller weights = less overfitting but more bias.

| α | CV RMSE |
|---|---|
| 1 | 0.8079 |
| 5 | 0.7503 |
| 10 | 0.7387 |
| **20** | **0.7353** |
| 50 | 0.7411 |
| 100 | 0.7515 |
| 200 | 0.7664 |

The loss curve is **U-shaped**:
- Low α (e.g. 1): underfitting is prevented but overfitting is severe → high RMSE
- High α (e.g. 200): model is over-regularised → collapses toward the intercept (user mean)
- Sweet spot: **α = 20**, flat valley between 10 and 50

---

### SVD Hyperparameter Sweep

Three axes: number of factors `f`, L2 regularisation `λ`, learning rate `η`.

**Key finding: Default Surprise settings (λ=0.02, η=0.005) are sub-optimal.**

| f | λ | η | CV RMSE |
|---|---|---|---|
| 25 | 0.02 | 0.005 | 0.8092 |
| 100 | 0.02 | 0.005 | 0.8148 ← more factors = WORSE with insufficient λ |
| 100 | 0.05 | 0.005 | 0.7976 |
| **100** | **0.05** | **0.010** | **0.7913** |
| 100 | 0.05 | 0.020 | 0.7948 |

**Insight:** At λ=0.02, increasing factors from 25→100 *raises* RMSE. More parameters overfit when not regularised enough. Once λ=0.05, the factor count matters less.

---

### SVD++ Sweep

| Axis | Grid | Best |
|---|---|---|
| Factors `f` | {25, 50, 100, 150} | **150** |
| Regularisation `λ` | {0.02, 0.03, 0.05, 0.07, 0.10} | **0.05** |
| Epochs | {20, 30, 40, 50} | **30** |

**Warning:** λ=0.02 with SVD++ causes RMSE=0.8281 (severe overfitting). The implicit feedback term `yⱼ` adds extra degrees of freedom that overfit to individual interaction sets.

**Convergence:** SVD++ converges quickly. 20 epochs ≈ 30 epochs; 50 epochs starts to overfit.

---

### Hybrid Alpha Sweep

| α (SVD weight) | CV RMSE |
|---|---|
| 0.1 | 0.7316 |
| **0.2** | **0.7308** |
| 0.3 | 0.7319 |
| 0.4 | 0.7349 |
| 0.5 | 0.7398 |
| 0.9 | 0.7776 |

The optimal blend is **80% content + 20% SVD**. Beyond α=0.4, SVD's weaker predictions progressively erode the content model's advantage.

---

## Why Specific Ranges Were Chosen

| Parameter | Range Logic |
|---|---|
| Ridge α ∈ {1, 5, 10, 20, 50, 100, 200} | Log-scale around the empirically expected value; covers 3 orders of magnitude |
| SVD f ∈ {25, 50, 100, 150, 200} | Sweet spot for ~180,000 ratings is around f=100; diminishing returns beyond 150 |
| SVD λ ∈ {0.02, 0.04, 0.05, 0.07, 0.10} | Fine-grained near 0.05 (known good value); coarser outside |
| k-NN k ∈ {20, 40, 80} | Literature range for MovieLens-scale datasets |

---

> [!WARNING]+ ❓ Assessor Question — How does the bias-variance trade-off apply to Ridge α?
> Low α → low bias (the model fits training data closely) but high variance (weights can become extreme, overfitting to the training fold). High α → high bias (weights are shrunk toward zero, losing personalisation) but low variance. The U-shaped RMSE curve on the validation set directly shows this trade-off: RMSE decreases as α rises from 1 (reducing overfitting), reaches a minimum at α=20 (optimal bias-variance balance), then increases again as α rises further (increasing underfitting/bias).

> [!WARNING]+ ❓ Assessor Question — Why did you use grid search instead of Bayesian optimisation?
> Grid search is fully reproducible and interpretable: we can see exactly which configurations were tested and why. The grid is small enough (< 50 configurations per family) to be exhaustive without a smart search. Bayesian optimisation would be preferable if the grid were large (> 200 configs) or expensive to evaluate. The main disadvantage of grid search is that it misses values *between* grid points — which is why we noticed the optimum at α=20 was missed by the original {0.1, 1, 10, 100, 1000} grid.

> [!WARNING]+ ❓ Assessor Question — Why does increasing SVD factors sometimes hurt performance?
> With insufficient regularisation (λ=0.02), more factors = more parameters to overfit. Each additional latent dimension can capture noise in the training ratings rather than genuine preference signal. The model memorises the training set without generalising. This is the classic **capacity vs. generalisation** trade-off in machine learning: adding model capacity only helps if regularisation keeps pace with it.

---

**See also:** [[Workflow/08_Results_and_Decision_Making]] | [[Workflow/07_Statistical_Assessment]] | [[Codes/09_Configs]]
