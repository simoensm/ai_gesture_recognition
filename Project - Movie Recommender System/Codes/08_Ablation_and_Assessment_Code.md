---
tags:
  - recommender-systems
  - code
  - python
  - ablation
  - statistical-testing
  - MLSMM2156
---
# Ablation Study & Statistical Assessment Code

**Files:** `recsys/ablation_study.py` and `recsys/statistical_assessment.py`

For the conceptual explanation, see [[Workflow/06_Ablation_Study]] and [[Workflow/07_Statistical_Assessment]].

---

## `ablation_study.py` — Structure

### Key Global Parameters

```python
N_FOLDS_FAST   = 5       # for SVD, Content, Hybrid
N_FOLDS_SLOW   = 3       # for UserBased, ItemBased (O(n²))
TOP_N          = 10      # top-N list size for diversity metrics
N_DIV_USERS    = 300     # users sampled for ILD/Coverage/Novelty
N_DIV_USERS_CB = 100     # fewer for ContentBased (slow per-call)
CAND_ITEMS_CB  = 600     # candidate items capped for ContentBased
RANDOM_SEED    = 42
```

### `_build_configs()` — Config Builder

```python
def _build_configs():
    for name, cls, params in EvalConfig.models:
        is_slow = issubclass(cls, (UserBased, ItemBased))
        is_cb   = issubclass(cls, (ContentBased, ContentBasedWithStats, ...))
        
        entry = {
            "n_folds":       N_FOLDS_SLOW if is_slow else N_FOLDS_FAST,
            "div_users":     N_DIV_USERS_CB if is_cb else N_DIV_USERS,
            "div_max_items": CAND_ITEMS_CB  if is_cb else None,
            ...
        }
```

Automatically derives fold count and diversity sampling parameters from the model class. Class inspection (`issubclass`) determines whether the model is "slow" or content-based.

---

### Diversity Metric Functions

#### `_ild(top_n_recs, max_users)` — Intra-List Diversity

```python
def _ild(top_n_recs, max_users):
    for uid in sampled_users:
        items = [iid for iid, _ in top_n_recs[uid]]
        vecs  = GENRE_MAT[idxs]           # genre one-hot vectors for each item
        norms = np.linalg.norm(vecs, axis=1, keepdims=True)
        normed = vecs / norms              # unit vectors
        sim    = normed @ normed.T         # cosine similarity matrix (k×k)
        k = len(idxs)
        upper = sim[np.triu_indices(k, k=1)]  # upper triangle (no diagonal)
        scores.append(1.0 - float(np.mean(upper)))   # 1 - mean_similarity = diversity
```

**Step by step:**
1. Get genre vectors for the top-10 items
2. Normalise to unit length (cosine similarity = dot product of unit vectors)
3. Compute all pairwise cosine similarities → k×k matrix
4. Take the upper triangle (avoid double-counting pairs i,j and j,i)
5. ILD = 1 − mean_similarity (diversity = inverse of similarity)

`np.triu_indices(k, k=1)`: returns the row and column indices of the strictly upper triangular part of a k×k matrix. The `k=1` offset excludes the diagonal (an item is 100% similar to itself).

#### `_novelty(top_n_recs, popularity, max_users)` — Self-Information

```python
def _novelty(top_n_recs, popularity, max_users):
    for uid in sampled_users:
        items = [int(iid) for iid, _ in top_n_recs[uid]]
        novs  = [-np.log2(popularity.get(i, eps) + eps) for i in items]
        user_novs.append(float(np.mean(novs)))
```

`popularity[i] = #raters(i) / n_users`. Popular films have high popularity → low `-log₂(p)` → low novelty score. Obscure films have low popularity → high `-log₂(p)` → high novelty.

#### `_partial_anti_testset(trainset, n_users, max_items_per_user)` — Anti-Testset Sampling

```python
def _partial_anti_testset(trainset, n_users, max_items_per_user):
    for u_inner in sampled:
        rated_inner = {iid for iid, _ in trainset.ur[u_inner]}
        unrated = [i for i in all_items if i not in rated_inner]
        if max_items_per_user:
            unrated = rng.sample(unrated, max_items_per_user)
        for i_inner in unrated:
            anti.append((u_raw, i_raw, global_mean))   # dummy true_r = global_mean
```

Builds a list of (user, unrated_item, global_mean) triples. The global_mean as `true_r` is just a placeholder — the anti-testset is only used to generate predictions (which we then rank), not to compute RMSE.

---

### `compute_error_analysis(all_preds, user_activity, item_pop_counts)` — Error Breakdown

```python
df = pd.DataFrame(all_preds, columns=["uid", "iid", "true_r", "est_r"])
df["residual"] = df["est_r"] - df["true_r"]

ea["bias"] = float(df["residual"].mean())           # systematic error
ea["residual_std"] = float(df["residual"].std())    # spread

# Per-rating RMSE
for r_val in [0.5, 1.0, ..., 5.0]:
    mask = df["true_r"] == r_val
    per_rating[r_val] = np.sqrt((df.loc[mask, "residual"]**2).mean())

# Rating grid
df["true_bin"] = df["true_r"].apply(lambda r: round(r*2)/2)
df["pred_bin"] = df["est_r"].apply(lambda r: round(max(0.5, min(5.0, r))*2)/2)
grid = df.groupby(["true_bin", "pred_bin"]).size().unstack(fill_value=0)
```

**The rating grid** is the regression equivalent of a confusion matrix. `true_bin` is the actual rating (rounded to nearest 0.5). `pred_bin` is the predicted rating (clipped to [0.5,5.0] and rounded). The diagonal shows exact-bucket hits; near-diagonal shows ±0.5 star errors.

---

## `statistical_assessment.py` — Structure

### Data Loading

```python
def _load_master_results(path):
    df = pd.read_csv(path)
    
    # Deduplication: if a model was re-run, keep the run with the most folds
    fold_counts = df.groupby(["run_id", "model_name"])["fold"].nunique()
    # Sort: most folds first, then most recent run_id (lexicographic)
    best_run = fold_counts.sort_values(...).drop_duplicates("model_name")
    df = df[df["run_id"] == df["model_name"].map(best_run)]
```

If you run the ablation twice, `master_results.csv` has duplicate rows. The deduplication keeps the run with the most fold observations per model (most complete run). Ties go to the most recent run_id.

---

### `_build_metric_matrix(df, metric)` — Pivot for Friedman

```python
def _build_metric_matrix(df, metric):
    pivot = df.pivot_table(index="fold", columns="model_name",
                           values=metric, aggfunc="mean")
    pivot = pivot.dropna()   # keep only folds where ALL models have results
    return pivot
```

Produces an N×k matrix (folds × models). Each cell is the metric value for that (model, fold) pair. Dropping NaN rows ensures all models have the same folds — required for the paired Friedman test.

---

### `run_friedman(matrix, lower_better)` — Main Test

```python
def run_friedman(matrix, lower_better, alpha=0.05):
    stat, p = friedmanchisquare(*[matrix[col].values for col in matrix.columns])
    
    ranks_per_fold = _rank_matrix(matrix, lower_better)
    mean_ranks = ranks_per_fold.mean()
    
    return {"statistic": stat, "p_value": p, "significant": p < alpha, "mean_ranks": mean_ranks}
```

`friedmanchisquare(*arrays)` from scipy takes multiple arrays (one per model, each of length N=folds) and computes the Friedman χ² statistic.

`_rank_matrix` assigns ranks within each fold:
- For RMSE (lower_better=True): rank 1 = smallest RMSE in that fold
- For ILD (lower_better=False): rank 1 = largest ILD in that fold

---

### Critical Difference Diagram (`_draw_cd_diagram`)

```python
def _draw_cd_diagram(mean_ranks, cd, title, ...):
    ax.set_xlim(x_max, x_min)   # REVERSED: rank 1 on the RIGHT (= best)
    
    # Draw a horizontal bar connecting models whose rank diff < CD
    i = 0
    while i < k:
        j = i
        while j + 1 < k and ranks[j+1] - ranks[i] < cd:
            j += 1
        if j > i:
            ax.hlines(bar_y, ranks[i], ranks[j], ...)  # clique bar
        i += 1
```

The diagram: models are placed along a horizontal axis (position = mean rank). The CD bracket shows the critical difference. A thick bar connecting two or more models means they are NOT significantly different (their rank gap < CD). Models without a connecting bar ARE significantly different.

---

> [!WARNING]+ ❓ Assessor Question — Why does `pivot_table` use `aggfunc="mean"` if each (model, fold) should have exactly one value?
> In practice, each (model_name, fold) combination should appear once in `all_results.csv`. But if the ablation was re-run or if there are duplicate rows from multiple runs, a (model, fold) pair might appear twice. `aggfunc="mean"` handles this by averaging duplicates, preventing pivot errors. After deduplication in `_load_master_results`, this should rarely matter, but it's a safety net.

> [!WARNING]+ ❓ Assessor Question — What does `friedmanchisquare(*arrays)` do?
> It takes k arrays (one per model), each of length N (number of folds). It computes the Friedman test statistic: for each fold, rank the models; then compare average ranks across folds. The `*` unpacks the list of columns into individual arguments. For example, with 3 models A, B, C and 5 folds: `friedmanchisquare(matrix['A'].values, matrix['B'].values, matrix['C'].values)` returns (χ², p-value).

---

**See also:** [[Workflow/06_Ablation_Study]] | [[Workflow/07_Statistical_Assessment]] | [[Codes/09_Configs]]
