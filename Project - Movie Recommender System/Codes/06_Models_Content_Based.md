---
tags:
  - recommender-systems
  - code
  - python
  - content-based
  - ridge
  - MLSMM2156
---
# Content-Based Models

**File:** `recsys/models.py` — `ContentBased` (line 1056), `ContentBasedWithStats` (line 1214), `ContentBasedPlus` (line 1367)

For feature construction details, see [[Codes/02_Feature_Engineering_Code]]. For feature performance results, see [[Workflow/04_Feature_Engineering]].

---

## The Core Idea

Forget user-user and item-item similarity. Instead, represent every movie as a feature vector **x_i** and learn a **per-user weight vector w_u** such that:

$$\hat{r}_{u,i} = \mathbf{w}_u^\top \mathbf{x}_i$$

This is a separate **regression model per user**. With 1,000 users, that's 1,000 Ridge models trained independently.

**Why per-user?** Because two users with identical genre preferences might weight `thought-provoking` vs. `action-packed` very differently. A global model would mix their signals. Per-user models learn individual taste profiles.

---

## `ContentBased`

### `__init__`

```python
def __init__(self, features_method, regressor_method, n_genome_components=150, alpha=19.5):
    import re
    m = re.search(r'_a(\d+(?:\.\d+)?)$', features_method or '')
    if m:
        self.alpha = float(m.group(1))
        features_method = features_method[:m.start()]
    ...
    self.content_features = build_content_features(features_method, n_genome_components)
```

**Alpha encoding in string:** You can pass `"genome_full_a25"` as `features_method` — the regex extracts `25` as the Ridge alpha. This allows specifying both the feature method and the alpha in a single string, which is convenient for the `EvalConfig.models` list.

**Features are built at `__init__` time**, not at `fit()` time. This means the feature matrix is computed once and reused across all K folds. Good for speed; but it means the static features (like `genome_full`) are built from all items regardless of the fold split.

---

### `_user_feature_matrix(u, bias_y=False)`

```python
def _user_feature_matrix(self, u, bias_y=False):
    df_user = pd.DataFrame(self.trainset.ur[u], columns=['item_id', 'user_ratings'])
    df_user['item_id'] = df_user['item_id'].map(self.trainset.to_raw_iid)
    df_user = df_user.merge(self.content_features, how='left', ...)
    X = df_user[feature_names].fillna(0).values
    y = df_user['user_ratings'].values
    
    if bias_y:
        y = y - self._mu - bu - bi_vals   # subtract biases → train on residuals
    
    return X, y
```

For user u:
1. Gets all (inner_item_id, rating) pairs from the training fold
2. Converts inner item IDs to raw movie IDs (`to_raw_iid`)
3. Joins with the content feature matrix on movie ID
4. Returns (X, y) where X = feature matrix of rated items, y = ratings

`bias_y=True` subtracts the Koren biases (μ + b_u + b_i) from each rating. The regressor then fits on the **residual** preference signal — what's left after accounting for global mean and user/item biases.

---

### `fit(trainset)` — Training All User Profiles

```python
def fit(self, trainset):
    self.user_profile = {u: None for u in trainset.all_users()}
    
    for u in self.user_profile:
        X, y = self._user_feature_matrix(u)
        self.user_profile[u] = Ridge(alpha=alpha).fit(X, y)
```

For each user, fit an independent Ridge regressor on their rated items. After fit, `user_profile[u]` holds the trained sklearn model for user u.

### Regressor Options

| Method | Code | When to use |
|---|---|---|
| `random_score` | Random uniform | Sanity check only |
| `random_sample` | Random from user's own ratings | Sanity check only |
| `linear_regression` | Unregularised OLS | Never (overfits severely) |
| `ridge` | `Ridge(alpha=α)` | Default — fast, robust |
| `ridge_cv` | `RidgeCV(alphas=[5, 10, 20, 25, 50, 100, 200])` | Per-user α tuning (no benefit over fixed α=20) |
| `ridge_bias` | Ridge on bias-corrected residuals | Marginal improvement |
| `ridge_adaptive` | α ∈ {100, 25, 10, 2} based on user activity | Helpful for sparse users |
| `random_forest` | RF(50 trees, max_depth=6) | Non-linear; rarely better, slower |
| `svr` | SVR(rbf, C=5, ε=0.2) | Slowest; rarely better |

`ridge_adaptive` logic:
- n < 10 ratings → α = 100 (heavy regularisation, sparse user)
- n < 50 → α = 25
- n < 200 → α = 10
- n ≥ 200 → α = 2 (light regularisation, power user)

---

### `estimate(u, i)` — Prediction

```python
def estimate(self, u, i):
    raw_item_id = self.trainset.to_raw_iid(i)
    item_features = self.content_features.loc[raw_item_id].values
    
    regressor = self.user_profile[u]
    score = float(regressor.predict([item_features])[0])
    
    if self.regressor_method.endswith('_bias'):
        score = self._mu + self._b_u[u] + self._b_i.get(i, 0.0) + score
    
    return float(np.clip(score, lo, hi))
```

For bias-corrected variants: the final prediction adds back the biases that were subtracted during training.

---

## `ContentBasedWithStats` ⭐ Best standalone model

Extends `ContentBased` by appending item quality statistics computed **from the trainset** (leak-free).

### Why This Is Architecturally Tricky

```python
class ContentBasedWithStats(ContentBased):
    def __init__(self, ...):
        ContentBased.__init__(self, ...)
        # Snapshot static features NOW — before any fold modifies them
        self._base_content_features = self.content_features.copy()
    
    def fit(self, trainset):
        # Reset to the snapshot on every fit() call
        self.content_features = self._base_content_features.copy()
        
        # Compute stats from THIS fold's training set (no leakage)
        stats = {}
        for i in trainset.all_items():
            ratings = [r for _, r in trainset.ir[i]]
            n = len(ratings)
            mu_i = float(np.mean(ratings))
            stats[trainset.to_raw_iid(i)] = {
                'item_bayesian_mean': (n * mu_i + 25 * global_mean) / (n + 25),
                'item_log_count': np.log1p(n),
                'item_std': np.std(ratings) if n > 1 else 0.0,
                'item_high_ratio': np.mean([r >= 4 for r in ratings]),
            }
        
        # Scale stats to [0,1] and append to content features
        df_stats = pd.DataFrame.from_dict(stats, orient='index')
        scaler = MinMaxScaler()
        df_stats_scaled = pd.DataFrame(
            scaler.fit_transform(df_stats), ...)
        
        self.content_features = pd.concat(
            [self.content_features, df_stats_scaled], axis=1
        ).fillna(0.5)
        
        return ContentBased.fit(self, trainset)
```

**The snapshot pattern:** Without `self._base_content_features = self.content_features.copy()` in `__init__`, each call to `fit()` would append 4 new stat columns to the existing feature matrix. After 5 folds, the matrix would have 20 extra stat columns instead of 4, completely corrupting predictions.

The snapshot ensures every `fit()` call starts from the original static features.

**Items absent from trainset:** Items in the genome matrix that didn't appear in this fold's training data get `0.5` (midpoint imputation) for their stat features.

---

## `ContentBasedWithStats100` and `ContentBasedWithStats150`

```python
class ContentBasedWithStats150(ContentBasedWithStats):
    def __init__(self, features_method, regressor_method):
        ContentBasedWithStats.__init__(self, features_method, regressor_method, n_genome_components=150)
```

Drop-in variants with fixed `n_genome_components`. Used by the hackathon submission interface where a simpler constructor signature is needed.

---

## `ContentBasedPlus`

Hardcoded to `content_based_plus` features (genome + visuals + item stats from full dataset):

```python
class ContentBasedPlus(ContentBased):
    def __init__(self, regressor_method='ridge'):
        self._static_features = build_content_features("content_based_plus")
    
    def fit(self, trainset):
        # Computes item stats from trainset (leak-free)
        # but static features include _item_stats_scaled (leaky from full dataset)
        ...
```

Less rigorous than `ContentBasedWithStats` — the static features already include item stats from the full dataset. Used only for serving in the webapp.

---

> [!WARNING]+ ❓ Assessor Question — Why fit a separate Ridge model per user instead of one global model?
> A single global model would learn average weights across all users: "genome dimension 42 (suspense) → positive weight". But individual users have very different preferences: one user loves suspenseful films, another avoids them. A global model can't capture this — it would assign suspense a moderate weight that satisfies neither user. Per-user models allow fully personalised weight vectors: user A's model might weight suspense at +2.1, user B's at -0.8.

> [!WARNING]+ ❓ Assessor Question — What happens for users with very few ratings (cold users)?
> For a user with n=5 ratings and a 1,128-dim feature vector, the Ridge regression system is severely underdetermined: 5 equations, 1,129 unknowns (1,128 weights + intercept). Ridge adds the constraint ||w||² ≤ budget, which provides enough regularisation to get a solution, but it converges toward an intercept-only prediction (essentially the user's mean rating). The 1,128-dim weight vector becomes near-zero, losing all personalisation. For cold users, the model degrades gracefully to a popularity-aware baseline rather than producing personalised recommendations.

> [!WARNING]+ ❓ Assessor Question — How does Ridge differ from ordinary least squares (OLS)?
> OLS minimises ||y − Xw||² with no constraints on w. For underdetermined systems (more dimensions than samples), OLS has infinitely many solutions and picks one with very large weights that overfit perfectly to the training data. Ridge adds the term α||w||² to the objective, forcing weights to be small. The solution is `w* = (XᵀX + αI)⁻¹ Xᵀy`. The αI term makes the matrix invertible even when XᵀX is rank-deficient (fewer samples than features) — this is the "well-posed" property that makes Ridge suitable for per-user regression with many features and few ratings.

---

**See also:** [[Codes/02_Feature_Engineering_Code]] | [[Codes/07_Models_Hybrid]] | [[Workflow/04_Feature_Engineering]]
