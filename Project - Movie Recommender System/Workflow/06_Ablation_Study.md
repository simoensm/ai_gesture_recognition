# Ablation Study Workflow

The ablation study is the engine that runs all model configurations systematically and produces all results.

**File:** `recsys/ablation_study.py` | Code details: [[Codes/08_Ablation_and_Assessment_Code]]

---

## What Is an Ablation Study?

An ablation study tests many model configurations systematically to understand which components matter and by how much. "Ablation" literally means removing components one at a time to see what breaks.

In our context: we test every combination of (model class × hyperparameters) and compare their RMSE, diversity, and error patterns.

---

## Input: `EvalConfig.models`

All configurations to test are registered in `configs.py` as a list of 3-tuples:

```python
(name, ModelClass, kwargs_dict)
```

Example:
```python
("cbs_genome_full_ridge_a20",
 ContentBasedWithStats,
 {"features_method": "genome_full", "regressor_method": "ridge", "alpha": 20})
```

This single source of truth means adding a new configuration requires only one line in `configs.py`. See [[Codes/09_Configs]].

---

## How It Runs

```
For each configuration in EvalConfig.models:
    1. Instantiate the model with its kwargs
    2. Run K-Fold CV (5 folds for most, 3 for slow CF models)
    
    For each fold:
        a. Fit on training fold
        b. Evaluate RMSE + MAE on test fold
        c. Build anti-testset → compute top-10 recommendations → ILD + Coverage + Novelty
        d. Collect raw predictions (uid, iid, true_r, est_r) for error analysis
        e. Append fold row to all_results.csv (written live, fold-safe)
    
    After all folds:
        a. Compute error analysis (bias, per-rating RMSE, per-user RMSE, rating grid)
        b. Write text report to reports/<name>.txt
```

---

## Output Files

All outputs go to `data/hackathon/evaluations/ablation_<TIMESTAMP>/`:

| File | Content |
|---|---|
| `all_results.csv` | Every (model, fold) row — written live so a crash doesn't lose progress |
| `fold_N.csv` | All models evaluated on fold N |
| `summary.csv` | Mean ± std per model (accuracy + diversity) |
| `stats_ready.csv` | Wide format: fold_1…fold_N RMSE per model → drop into Wilcoxon notebook |
| `reports/<name>.txt` | Full text report per model: CV summary, per-fold breakdown, error analysis, rating grid |

Persistent (append-only across ALL runs):
| File | Content |
|---|---|
| `master_results.csv` | All fold rows from all ablation runs combined |
| `master_stats_ready.csv` | All stats_ready rows combined |

The `master_results.csv` feeds directly into [[Workflow/07_Statistical_Assessment]].

---

## Error Analysis (Per Model)

For each configuration, the ablation computes a detailed error breakdown over all folds' predictions:

### Residual Statistics
- **Bias** = mean(predicted − true): positive = systematic over-prediction
- **Residual std**: spread of errors
- **Percentiles**: p10, p25, p50, p75, p90 of residuals

### Per-True-Rating RMSE
RMSE computed separately for each rating bucket (0.5, 1.0, 1.5, …, 5.0). This reveals where the model struggles — typically low ratings are harder to predict because users rarely give 1-star ratings.

### Per-User Activity RMSE
Users split into 4 quartiles by their number of training ratings:
- Q1 (sparse users): few ratings → harder to personalise
- Q4 (power users): many ratings → easier to build accurate profiles

### Per-Item Popularity RMSE
Items split into 4 quartiles by their total rating count:
- Q1 (niche): few raters → unreliable item signal
- Q4 (popular): many raters → stable statistics

### Rating Grid
A 10×10 matrix where `grid[true_bucket][predicted_bucket]` counts predictions. The recommender equivalent of a confusion matrix. Measures:
- **Diagonal accuracy**: fraction of predictions in the exact correct bucket
- **Near-diagonal accuracy**: fraction within ±0.5 stars

---

## Latency Metrics

The ablation also measures **serving speed** (important for production):

| Metric | Description |
|---|---|
| `fit_time` | Training time in seconds |
| `pred_ms_per_rating` | Milliseconds to predict one (user, item) rating |
| `rec_ms_per_user` | Milliseconds to generate top-10 for one user |

Content-based models are slower per-call because they run a forward pass through a pandas DataFrame for each item. The hybrid model combines both.

---

> [!WARNING]+ ❓ Assessor Question — How is the ablation fold-safe?
> The script writes each (model, fold) row to `all_results.csv` immediately after that fold completes, using `mode="a"` (append). If the script crashes mid-run, all completed fold rows are preserved. The `master_results.csv` also accumulates results across separate runs, so re-running after a crash doesn't overwrite earlier work.

> [!WARNING]+ ❓ Assessor Question — What is the anti-testset and why is it needed?
> The test set contains (user, item, true_rating) triples — it's used for RMSE computation. But to compute *recommendation lists* for ILD/Coverage/Novelty, we need to score all *unrated* items for each user. This is the anti-testset: for each user, all items they haven't rated in the training fold. We then run the model on this set and take the top-10 predictions as the recommendation list.

> [!WARNING]+ ❓ Assessor Question — Why are content-based models sampled to 100 users for diversity metrics?
> Content-based models compute predictions item by item via `pandas.loc` lookup for each (user, item) pair. Scoring 600 candidate items for 300 users = 180,000 forward passes, each with a DataFrame lookup. This takes several minutes per fold. Using 100 users × 600 items instead makes it feasible while still computing a meaningful diversity estimate (~10% of users).

---

**See also:** [[Codes/08_Ablation_and_Assessment_Code]] | [[Workflow/07_Statistical_Assessment]] | [[Workflow/08_Results_and_Decision_Making]]
