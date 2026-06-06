# Feature Engineering Code

**File:** `recsys/models.py` — the `build_content_features()` factory function and its helpers.

For the conceptual explanation of each feature method, see [[Workflow/04_Feature_Engineering]].

---

## Architecture: The Factory Pattern

```python
def build_content_features(features_method, n_genome_components=150):
    df_items = load_items()        # Always loaded first
    
    if features_method == "genres":
        df_features = _genre_onehot(df_items)
    elif features_method == "genome":
        df_features = _genome_svd_features(df_items, n_genome_components)
    # ... 20+ elif branches
    
    return df_features             # Always a DataFrame: index=movieId, columns=feature_names
```

All feature methods return the **same structure**: a pandas DataFrame with `movieId` as the index. This uniform interface means any content-based model can accept any feature method without code changes — just pass a different string.

---

## Helper Functions

### `_genre_onehot(df_items)`
```python
def _genre_onehot(df_items):
    df = df_items[C.GENRES_COL].str.get_dummies(sep='|')
    return df.drop(columns=['(no genres listed)'], errors='ignore')
```

`str.get_dummies(sep='|')` splits the genre string `"Action|Adventure|Sci-Fi"` on the pipe character and creates binary columns. One-liner that pandas makes elegant.

---

### `_year_features(df_items)`
```python
def _year_features(df_items):
    raw = df_items[C.LABEL_COL].str.extract(r'\((\d{4})\)', expand=False).astype(float)
    return raw.fillna(raw.median()).rename('release_year').to_frame()
```

`str.extract(r'\((\d{4})\)')` uses a **regex** to find a 4-digit year wrapped in parentheses inside the title string (e.g. `"Toy Story (1995)"` → `1995`). `fillna(raw.median())` handles the rare movies with no year in the title.

---

### `_genome_svd_features(df_items, n_components)` ⭐
```python
def _genome_svd_features(df_items, n_components):
    df_pivot = load_genome()                                    # movieId × tagId matrix
    n = min(n_components, df_pivot.shape[1])                   # don't request more dims than exist
    reduced = TruncatedSVD(n_components=n, random_state=42).fit_transform(df_pivot.values)
    df = pd.DataFrame(reduced, index=df_pivot.index, columns=[f'g{i}' for i in range(n)])
    df = df.reindex(df_items.index)                            # align to all movies
    return df.fillna(df.mean())                                # mean-impute missing movies
```

**`TruncatedSVD`:** Like PCA but works on sparse matrices efficiently. Given a matrix M, it finds the top-n directions of maximum variance. Applied to the 1,128-dim genome vectors, it produces n-dim compressed representations.

**`reindex(df_items.index)`:** Some movies in `df_items` are not in the genome pivot (they never appeared in the genome file). `reindex` adds NaN rows for them, which `fillna(df.mean())` then replaces with column averages.

---

### `_item_stats_scaled(df_items)` — ⚠️ Leakage Warning
```python
def _item_stats_scaled(df_items):
    """
    WARNING — DATA LEAKAGE: stats are computed from load_ratings(),
    which returns the complete CSV including any held-out test fold.
    Do NOT use inside a cross-validation loop.
    """
    df_r = load_ratings()
    ...
```

This function exists only for **offline feature exploration**. For CV-safe item statistics, use `ContentBasedWithStats.fit()` instead (which recomputes stats from `trainset.ir`).

---

## Key Feature Methods in Detail

### `"genome_full"` — Best single feature

```python
elif features_method == "genome_full":
    df_pivot = load_genome()
    df_features = df_pivot.reindex(df_items.index).fillna(df_pivot.mean())
```

No transformation — just the raw 1,128-dim relevance scores, mean-imputed. Ridge handles the high dimensionality via L2 regularisation.

---

### `"cf_svd"` — Collaborative features used as content

```python
elif features_method == "cf_svd":
    df_r = load_ratings()
    R = (
        df_r.pivot(...)         # users × items rating matrix
        .sub(global_mean)       # mean-center: remove popularity axis
        .fillna(0)              # treat unrated as "no evidence"
    )
    item_factors = TruncatedSVD(n_components=n).fit_transform(R.values.T)
    # R.T = items × users, so fit_transform gives item factors (items × k)
```

**Insight:** By transposing R before SVD, we decompose the *item-by-user* matrix. The left singular vectors (after scaling by Σ) become **item latent factors** — they encode "what collective rating pattern this movie generates." These are then used as features in a per-user Ridge regressor.

**Minor leakage note:** Item factors are computed from the full ratings matrix (train + test). Each user contributes ~0.1% to each item factor, so bias is negligible.

---

### `"genre_genome_weighted"` — Manual budget balancing

```python
elif features_method == "genre_genome_weighted":
    GENRE_SCALE = 30.0
    df_genre = _genre_onehot(df_items) * GENRE_SCALE
    df_features = pd.concat([df_genre, df_genome], axis=1).fillna(0)
```

Without scaling, Ridge gives 59× more "weight budget" to the 1,128-dim genome block than the 19-dim genre block. Multiplying genre by 30 equalises the L2 budget per feature block.

---

### `"content_based_plus"` — 3-signal combination

```python
elif features_method == "content_based_plus":
    # genome_full (1128) + visuals (7) + item_stats (5) — all merged
    df_features = pd.concat([df_genome, df_vis, _item_stats_scaled(df_items)], axis=1).fillna(0.5)
```

Note: uses `_item_stats_scaled` which is leaky — only suitable for offline exploration. The CV-safe version is `ContentBasedWithStats`.

---

## Why `fillna(0.5)` vs `fillna(df.mean())`?

- `fillna(df.mean())` fills with the feature column average. Appropriate when 0 would be misleading (e.g. genome scores — a missing score isn't "zero relevance").
- `fillna(0.5)` fills with a midpoint. Used in multi-source combinations where 0.5 represents "unknown, assume average."
- `fillna(0)` used for binary features (director/cast one-hot) — a missing actor simply wasn't in that movie.

---

> [!WARNING]+ ❓ Assessor Question — What does `TruncatedSVD` do differently from a full SVD?
> Full SVD decomposes a matrix M = U × Σ × Vᵀ completely, computing all singular vectors. For a 1,128-dimensional genome matrix, that's 1,128 singular vectors. TruncatedSVD only computes the top-n singular vectors — the ones corresponding to the largest singular values (most variance). It uses the Lanczos algorithm, which is much faster for sparse/tall matrices and doesn't require computing the full decomposition. For our genome matrix, computing only the top 100 SVD components is ~10× faster than full SVD.

> [!WARNING]+ ❓ Assessor Question — What is `pd.concat([...], axis=1)` doing?
> `pd.concat(axis=1)` stacks DataFrames side by side (column-wise), aligning them on their index (movieId). If a movie appears in genome but not in visuals, `pd.concat` will insert NaN for the missing visual columns. The subsequent `.fillna(0.5)` handles these NaNs. This is how we build combined feature matrices from multiple sources, each with different movie coverage.

---

**See also:** [[Workflow/04_Feature_Engineering]] | [[Codes/06_Models_Content_Based]] | [[Codes/01_Constants_and_Loaders]]
