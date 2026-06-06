# Server Endpoints

**File:** `recsys/server.py` — `recommend()` and `similar()` functions

For server architecture and startup, see [[Webapp/01_Webapp_Overview]].

---

## `POST /recommend` — The Main Endpoint

### Request Body

```json
{
  "watched":     [movie_id, ...],
  "ratings":     {"movie_id": 4.0, "movie_id2": 3.5},
  "preferences": {
    "genres":      ["Action", "Drama"],
    "directors":   ["Christopher Nolan"],
    "actors":      ["Heath Ledger"],
    "genome_tags": ["plot_twist", "suspense"],
    "era":         "2000s"
  },
  "n": 200
}
```

- `watched`: movies seen but not rated → assigned implicit rating 3.5
- `ratings`: explicit ratings (override watched)
- `preferences`: onboarding preferences used by Phases 0 and 1
- `n`: number of recommendations to return (default 200)

### Response Body

```json
{
  "recommendations": [movie_id, ...],
  "matchScores":     {"movie_id": 87},
  "movieTags":       {"movie_id": ["suspenseful", "atmospheric"]},
  "topUserTags":     ["suspenseful", "thought-provoking", "atmospheric"],
  "coldStart":       false,
  "phase":           2,
  "model":           "Hybrid · SVD + Genome Ridge"
}
```

`matchScores`: integers 0–100 (percentage). Computed from `(prediction − 0.5) / 4.5 × 100`.

`movieTags`: genome tags that explain why each movie was recommended. Empty for CF models (no Ridge coefficients). See [[Webapp/01_Webapp_Overview]].

---

## The Cascade Hybrid (Burke 2002)

The `/recommend` logic implements a **cascade hybrid** with 3 phases, determined by how many ratings the new user has provided:

```
n_actual = 0       → Phase 0  (cold start)
0 < n_actual < 20  → Phase 1  (grey start — progressive blend)
n_actual ≥ 20      → Phase 2  (full configured model)
```

---

### Phase 0 — Cold Start (0 ratings)

```python
pseudo   = _prefs_to_pseudo_ratings(prefs, rated_set)
cold_ids = [mid for mid in pseudo if mid in _cold_id_to_idx]

if len(cold_ids) >= 3:
    X_cold = _cold_feat_matrix[[_cold_id_to_idx[mid] for mid in cold_ids]]
    y_cold = np.array([pseudo[mid] for mid in cold_ids])
    reg0   = Ridge(alpha=20.0)
    reg0.fit(X_cold, y_cold)
    raw = np.clip(reg0.predict(_cold_feat_matrix), 0.5, 5.0)
    # + era boost (soft, +0.2 for preferred decade)
```

**What is `_prefs_to_pseudo_ratings()`?**

Converts stated preferences into synthetic ratings. For each movie in the catalog:
```
score = 0
score += min(genre_hits × 0.8, 2.0)   # up to 3 genre matches
score += 1.5 if director matches
score += min(actor_hits × 0.8, 1.6)   # up to 2 actor matches
score += 0.6 if genome anchor (tag relevance > 0.80)
score += 0.3 if year in preferred era  (soft signal)
```
Then normalises: `rating = 3.5 + 1.5 × (score / max_score)` → range [3.5, 5.0].

Movies with score ≤ 0.5 are excluded (no match at all).

**Why ≥ 3 anchors?** A Ridge regressor fitted on fewer than 3 examples can't learn meaningful weights. If there aren't enough preference matches, the server falls back to genre-filtered popularity.

**Fallback (< 3 anchors):** Filter `_fallback_order` (popularity-sorted) by the user's preferred genres, then return the top n.

---

### Phase 1 — Grey Start (1–19 ratings)

```python
alpha   = n_actual / 20.0          # 0.05 (1 rating) → 0.95 (19 ratings)
blended = alpha * content_scores + (1.0 - alpha) * _pop_scores_arr
```

As ratings accumulate, the blend shifts from predominantly popularity-driven toward predominantly content-personalised. The content model is always fitted on `{pseudo-ratings} ∪ {actual ratings}`, so stated preferences keep contributing until the user has ≥ 20 actual ratings.

`_pop_scores_arr`: pre-computed array of popularity scores aligned to `_cold_movie_ids`. Built at startup from the movies.json ranking: rank-0 → 5.0, not-in-list → 0.5. No per-request computation needed.

---

### Phase 2 — Warm Start (≥ 20 ratings)

The configured model from `WebappConfig.MODEL_CLASS` takes over fully. The code branches per model class:

#### HybridSVD (default)

```python
# SVD arm: fold new user into pre-trained item factors
Q_rated    = _svd_item_factors[rated_idx]         # item factor rows for rated movies
b_rated    = _svd_item_biases[rated_idx]          # item biases for rated movies
y_svd      = y_vals - _global_mean - b_rated      # residuals after global mean + item bias
lam        = max(0.02, 1.0 / n_rated)             # adaptive λ: few ratings → more regularisation
reg_svd    = Ridge(alpha=lam)
reg_svd.fit(Q_rated, y_svd)
p_u        = reg_svd.coef_                        # new user's latent vector (100-dim)
b_u        = reg_svd.intercept_                   # new user's bias
svd_scores = global_mean + b_u + b_i + Q @ p_u   # score for all items

# Content arm: per-user Ridge on genome features
X_cb       = _feat_matrix[rated_idx]
reg_cb     = Ridge(alpha=20.0)
reg_cb.fit(X_cb, y_vals[:n_rated])
cb_scores  = reg_cb.predict(_feat_matrix)

# Blend
all_scores = 0.2 * svd_scores + 0.8 * cb_scores
```

**The key insight for SVD at serving time:** The item factor matrix Q and item biases b_i are fixed (trained at startup on all data). For the new user, we only need to estimate `p_u` (user latent vector) and `b_u` (user bias). This is solved by Ridge regression: given the rated items' latent vectors Q[rated] and the user's ratings y, find p_u that minimises ||y − (μ + b_rated + Q[rated]·p_u)||² + λ||p_u||². This is a simple closed-form regression problem — much faster than re-training the full SVD.

#### UserBased

```python
new_centered = user's ratings - user's mean (fill unrated with 0)
sims         = _R_norm @ new_centered / ||new_centered||    # cosine with all training users
top_k        = argsort(sims)[-K:]                           # K most similar
all_scores   = new_mean + Σ(sim × (rating - neighbor_mean)) / Σ(sim)
```

#### ItemBased

```python
sims      = _item_factors_norm @ rated_factors.T   # pre-computed cosine via SVD
# keep only top-K per item, zero others
all_scores = item_means + (sims × y_centered).sum(axis=1) / sims.sum(axis=1)
```

---

## `POST /similar` — Item Similarity

```json
Request:  {"movie_id": 123, "n": 20}
Response: {"results": [{"id": 456, "similarity": 0.923}, ...], "method": "..."}
```

Computes cosine similarity between the query movie and all other movies in the active feature space:

```python
query_vec  = _feat_matrix[idx]                     # feature vector of query movie
sims       = (_feat_matrix @ query_vec) / (row_norms × ||query_vec||)
sims[idx]  = -1.0                                  # exclude self
top_idx    = argsort(sims)[-n:][::-1]
```

The feature space used depends on the active model:
- ContentBased/HybridSVD → genome features (1,128 dims)
- SVD → item latent factors (100 dims)
- ItemBased → SVD-approximated item-cosine space (100 dims)
- UserBased → normalised user-rating vectors per item

---

> [!WARNING]+ ❓ Assessor Question — Why does the adaptive λ = `1 / n_rated` for the SVD user projection?
> When fitting the new user's latent vector p_u via Ridge, the optimal regularisation strength λ depends on how much data we have. With only 1 rating, the system is wildly underdetermined: 100 unknowns (p_u dimensions) and 1 equation. Strong regularisation (λ large) is needed to prevent the solver from returning an extreme solution. With 50 ratings, the system is better-constrained and lighter regularisation is appropriate. `λ = 1/n_rated` implements this heuristic automatically: λ=1.0 at n=1 (strong regularisation), λ=0.05 at n=20 (light regularisation). The `max(0.02, ...)` floor prevents division-by-zero and ensures a minimum regularisation is always applied.

> [!WARNING]+ ❓ Assessor Question — Could the recommendation server serve thousands of users simultaneously?
> Not as implemented. The server is single-threaded (`debug=False`, single Flask worker). Each `/recommend` call is synchronous and blocks while fitting Ridge, running matrix multiplications, and sorting results. For concurrent users, you would need: (1) a multi-process WSGI server (e.g. Gunicorn with N workers), (2) request queueing if compute is shared, (3) caching of user profiles for repeat visitors. The current design is appropriate for a demo/hackathon with a few dozen concurrent users at most.

> [!WARNING]+ ❓ Assessor Question — What is the difference between Phase 0 and Phase 1 content model?
> Both phases fit the same Ridge regressor on genome features. The difference is the training data: Phase 0 uses only **pseudo-ratings** derived from stated preferences (no actual ratings yet). Phase 1 **merges** pseudo-ratings with actual ratings, with actual ratings taking precedence (`{**pseudo, **dict(rated_pairs)}`). Additionally, Phase 1 blends the content scores with popularity: as n_actual grows from 1 to 19, popularity influence decreases from 95% to 5%. Phase 2 drops popularity entirely and uses only the configured model.

---

**See also:** [[Webapp/01_Webapp_Overview]] | [[Codes/07_Models_Hybrid]] | [[Codes/09_Configs]]
