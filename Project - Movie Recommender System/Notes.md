
How do we choose k and the similarity metric ?
How do we handle negative similarities ?
How do we handle missing ratings ?
How do we handle no/small overlap ?

## 1. How do we choose k and the similarity metric?

### The similarity metric: Pearson correlation

Your model computes similarity this way ([server.py:1073](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1073)):

```python
new_centered = np.where(rated_mask, new_vec - new_mean, 0.0)
new_normalized = new_centered / np.linalg.norm(new_centered)
sims = _R_norm @ new_normalized   # dot product of two normalized, mean-centred vectors
```

This is **Pearson correlation**. The math:

```
sim(u, v) = Σ (r_ui - μ_u)(r_vi - μ_v)
            ─────────────────────────────────────
            √Σ(r_ui - μ_u)² · √Σ(r_vi - μ_v)²
```

Subtracting the mean before dividing by the norm = computing cosine similarity on mean-centred vectors. These are mathematically identical.

**Why Pearson and not cosine directly?** Because raw cosine ignores rating bias. If user A rates everything 4-5 and user B rates everything 1-2, they have opposite tastes — but their raw rating vectors are nearly parallel (both are "positive"). Mean-centring removes that bias. After centring, what matters is the _shape_ (did both go up on action films, down on romance?) not the scale.

**Why not Spearman?** Your ablation tested both ([ablation_study.py:186-191](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/ablation_study.py#L186-L191)):

- Spearman k=40: RMSE **0.8335**
- Pearson k=80: RMSE **0.8280** ← best

Spearman converts ratings to ranks before computing correlation. It's more robust to outliers but loses magnitude information. For this dataset, Pearson wins.

### The k value: k=80

You tested k ∈ {20, 40, 80} ([ablation_study.py:165-167](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/ablation_study.py#L165-L167)):

|k|Pearson RMSE|
|---|---|
|20|0.8390|
|40|0.8340|
|80|**0.8280** ← chosen|

**Why does larger k help here?** Your dataset has ~1,000 users × 8,737 items at 4.36% density. For any given movie, only ~43 users have rated it. With k=20, you might not find enough users who rated _that specific movie_ among the top-20 neighbors — so predictions fall back to item means too often. With k=80 you cast a wider net, increasing the chance that at least a few neighbors have seen the target movie. The law of diminishing returns kicks in beyond that — k=160 would give marginal gain but add noise from weakly similar users.

---

## 2. How do we handle negative similarities?

In code ([server.py:1077-1080](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1077-L1080)):

```python
pos_mask = top_sims > 0
if not pos_mask.any():
    all_scores = np.clip(_item_means_ub, 0.5, 5.0)   # fallback
```

**Negative similarity = anti-correlated user.** If sim(u, v) = −0.7, it means: when v liked a movie, you tended to dislike it. Including negative similarities in the weighted average would _invert_ their contribution — their positive ratings would pull your prediction down, their negative ratings would pull it up. You could exploit this (negative sims = inverted signal = still information), but it adds complexity and can destabilise predictions for sparse data. **The safe choice: discard them.** Your ablation confirmed this works well.

**Denominator guard** ([server.py:1095](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1095)):

```python
denom = (w * rated_by_nbr).sum(axis=0) + 1e-8
```

The `1e-8` prevents division by zero if all `pos_sims` are zero after masking.

---

## 3. How do we handle missing ratings?

Two distinct "missing" cases:

**Case A: A neighbor hasn't rated a movie** — handled here ([server.py:1092](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1092)):

```python
centered_nbr = np.where(np.isnan(centered_nbr), 0.0, centered_nbr)
rated_by_nbr = (~np.isnan(pos_ratings)).astype(np.float32)  # 0 or 1
```

When computing `Σ sim·(r_vi - μ_v)` in the numerator, a neighbor who hasn't rated movie i contributes **0** (not their mean, not a global mean — literally zero). The denominator only sums `sim_v` for neighbors who _have_ rated the movie (`rated_by_nbr` is 0 for unseen). So a missing rating contributes nothing to either side of the fraction. This is correct: "no evidence" ≠ "evidence of mediocrity".

**Case B: Computing the active user's similarity** — handled by `new_centered` ([server.py:1066](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1066)):

```python
new_centered = np.where(rated_mask, new_vec - new_mean, 0.0)
```

Same idea: movies the active user hasn't rated get 0 in the centred vector. They don't contribute to the dot product with any neighbor's centred vector.

**Mathematical interpretation:** `fillna(0.0)` after mean-centring = "this user/item pair is neutral, I have no signal." It's equivalent to assuming any unseen rating would be exactly average.

---

## 4. How do we handle no/small overlap?

**No overlap at all** — the `pos_mask` fallback ([server.py:1079-1080](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1079-L1080)):

```python
if not pos_mask.any():
    all_scores = np.clip(_item_means_ub, 0.5, 5.0)
```

If zero neighbors have positive similarity with the active user (extreme edge case — e.g. a user who rated only 1-2 obscure movies nobody else touched), predictions fall back to item-level popularity means.

**Small overlap — handled by `min_support`** in the ablation ([ablation_study.py:158](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/ablation_study.py#L158)):

```python
sim_options = {"name": "pearson", "min_support": 3}
```

`min_support=3` means: a similarity is only considered valid if users u and v have rated **at least 3 movies in common**. With fewer than 3 co-ratings, the Pearson correlation is unreliable (one outlier rating can flip it from +1 to -1). If the overlap is too small, that neighbor is excluded entirely.

**Why 3?** With 2 co-ratings you have exactly zero degrees of freedom for Pearson — correlation is always ±1. Three is the practical minimum for a meaningful estimate. You could raise it to 5 for higher reliability at the cost of fewer usable neighbors.

---

**To summarise the design decisions:**

|Problem|Your solution|Why|
|---|---|---|
|Which metric?|Pearson (cosine on centred)|Removes rating bias; ablation win|
|Which k?|80|Sparse data; more neighbors = better coverage|
|Negative sims|Discard (`pos_mask > 0`)|Inverted signal is destabilising|
|Missing ratings|`fillna(0)` after centring|"No evidence" = neutral contribution|
|No overlap|Fallback to item means|Can't trust similarity with zero co-ratings|
|Tiny overlap|`min_support=3`|Pearson undefined with <3 co-ratings|




Why can we not precompute every prediction during the offline phase ?
### The core reason: the active user doesn't exist at training time

Look at what happens at inference ([server.py:1058-1066](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1058-L1066)):

```python
new_vec = np.zeros(len(_movie_ids), dtype=np.float32)
for mid, r in user_ratings.items():         # ← ratings from THIS session
    new_vec[_id_to_movie_idx[mid]] = r

new_mean     = float(new_vec[rated_mask].mean())
new_centered = np.where(rated_mask, new_vec - new_mean, 0.0)
new_norm     = float(np.linalg.norm(new_centered))
sims         = _R_norm @ new_normalized     # ← compare to ALL training users
```

The `user_ratings` dict is built from what you typed into the webapp **right now** ([server.py:815-826](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L815-L826)). The active user is not one of the 1,000 training users — they are a new person arriving with unknown preferences. You cannot precompute `new_vec` because it doesn't exist until they rate something.

---

### The four concrete blockers

**1. The active user is anonymous until they interact**

You have `_R_raw` (1,000 × 8,737 matrix, precomputed). The 1,000 training users are fixed. The webapp user is _user 1,001_ — they arrive empty-handed. Their preference vector only materialises as they rate movies during the session. There is no row to precompute.

**2. The preference vector changes with every rating**

Each new rating shifts `new_mean` and reshapes `new_centered`, which recomputes every similarity score:

```
sims = _R_norm @ new_normalized
```

After rating 3 movies, you have one similarity ranking. After rating 4 movies, every single similarity score changes (because the mean shifted). You'd have to recompute and store a full ranking for every possible rating history — that's combinatorially impossible.

**3. Scale explodes even for known users**

Even if you only served the 1,000 training users: 1,000 users × 8,737 items = **8.7 million predictions**. Each changes every time any user adds a rating (because `_R_raw` would update, shifting neighbours' means). In production — Netflix has 230M users, 20,000+ titles — that's **4.6 trillion** predictions to precompute and keep fresh.

**4. Predictions are ordered by _un-watched_ items, which changes too**

Your server excludes already-rated movies from recommendations ([server.py:1116](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1116)):

```python
cand_item_indices = [i for i, mid in enumerate(_movie_ids) if mid not in rated_set]
```

So the ranked list for user u after watching movie A is different from the list after watching movie B. Each watch event invalidates the precomputed ranking.

---

### What _can_ be precomputed (and is, in your system)

|Artifact|Precomputed?|Why it's safe|
|---|---|---|
|`_R_norm` — normalised training matrix|✅|Training users' centred vectors never change|
|`_item_means_ub` — item-level fallback means|✅|Static aggregate|
|`_user_means_arr` — each training user's mean|✅|Static|
|Item factors (SVD)|✅|Trained once offline|
|Genome tag matrix|✅|Content features are static|
|**Active user similarity vector**|❌|Unknown until session starts|
|**Per-session ranked list**|❌|Depends on watch history|

The split: **anything about the training corpus** can be precomputed. **Anything about the active user** must be computed online. Your system does exactly this — the expensive matrix `_R_norm` (1,000 × 8,737, float32 ≈ 33 MB) is loaded once at startup, and the online cost is a single matrix-vector multiply `sims = _R_norm @ new_normalized`, which is O(users × items) ≈ 8.7M multiplications — fast enough to be imperceptible at serving time.

## Memory-based CF: Pros and Cons

"Memory-based" means you store the entire rating matrix and compute similarities directly at prediction time — no learned parameters, no training in the traditional sense. Your two methods are UserBased and ItemBased.

---

### Shared pros (both methods)

**Transparent and interpretable** You can always explain _exactly_ why a prediction was made. Your webapp shows this directly — the peer group, the neighbor contribs, the formula inputs. No black box: `r̂ = μ_u + Σ sim·(r_vj − μ_v) / Σ sim`.

**No training phase** Fit time is essentially loading the rating matrix into RAM. When a new user rates a movie, you don't retrain anything — you just recompute similarities on the fly. This is why your server loads in milliseconds.

**No overfitting in the classical sense** There are no learnable weights to overfit. The only hyperparameters are k and `min_support`, both regularising the neighbourhood size, not a high-dimensional parameter space.

**Domain-agnostic** The same algorithm works for movies, books, products, music — no domain knowledge required. You only need the rating matrix.

---

### Shared cons (both methods)

**Scalability is O(users × items) at prediction time** As explained earlier, you cannot fully precompute because the active user is new. At serving time your code does:

```python
sims = _R_norm @ new_normalized   # 1,000 dot products across 8,737 items
```

That's fine at 1,000 users. At 10M users this matrix multiply becomes intractable.

**Cold start — new items** A brand new movie has zero ratings → zero similarities → cannot be recommended by either method. Your system handles this with the Phase 0/Phase 1 cascade (popularity + genre blend), but that's a workaround, not a solution within CF.

**Sparsity kills accuracy** Your dataset is 4.36% dense. For any given (user, item) pair, you typically find ~1.9 users who rated _both_ movies in a pair. Similarities computed on 1-2 co-ratings are statistically meaningless. This is the dominant source of error in your RMSE.

**No latent structure** Memory-based methods treat each rating independently. They cannot discover that "Alien" and "Prometheus" are similar because both belong to the "sci-fi horror" latent dimension — they only know this if enough users happened to rate both. SVD/MF discovers these latent dimensions from the global signal.

---

### User-based specific

**Pro — Personalisation captures taste drift** User preferences evolve. If a user starts rating horror films highly, their similarity vector shifts immediately and they get pulled toward other horror fans — no retraining needed. The neighbourhood is recomputed live.

**Pro — Handles niche users well when peers exist** If you find even 3-4 users with very high Pearson correlation (0.7+), their shared niche taste is well represented, even if it's completely outside mainstream popularity.

**Con — User similarity is unstable** With 1,000 users and 8,737 items at 4.36% density, two users share on average ~38 co-rated movies. Pearson on 38 items is reasonable but fragile — a few extreme ratings can flip the similarity sign. This is why you saw RMSE std of ±0.0023 at k=80 (higher variance than SVD).

**Con — "Popular users" dominate** Users who rated 400+ movies have dense vectors — their similarities are stable and they appear as neighbours frequently. Users with 20 ratings have sparse, noisy vectors and are rarely selected as useful neighbours even if their taste actually matches. Your system partially mitigates this with `nRatings` shown in the peer group.

**Con — Slow at scale for exact similarity** Finding the top-k users requires comparing the active user's vector against all 1,000 training users. With 1M users this would need approximate nearest neighbour (ANN) methods like FAISS or HNSW.

---

### Item-based specific

**Pro — Item similarities are more stable** Items don't change their "nature" over time (The Godfather is always The Godfather). Item-item similarities computed once stay valid for months. In contrast, a user's tastes can shift week to week.

**Pro — Better suited to precomputation** Because the item catalogue is static, the full item-item similarity matrix (8,737 × 8,737 ≈ 305M pairs) _can_ be precomputed offline and cached. Your `ItemBased.estimate()` computes similarities on-demand during CV, but a production system would precompute the top-k similar items per item.

**Con — Sparsity hits harder** In your dataset, items have on average 37 ratings each. For two items to have a similarity computed, they need `min_support=3` co-raters. With 1,000 users and 8,737 items, the expected number of users who rated both item i and item j is:

```
E[co-raters(i,j)] = n_users × P(user rated i) × P(user rated j)
                  = 1000 × (37/8737) × (37/8737) ≈ 0.017
```

**Less than 1 expected co-rater per item pair.** This is why ItemBased is essentially broken on your dataset without SVD approximation — there simply aren't enough co-ratings to compute meaningful item-item similarities directly.

**Con — Cannot capture taste profiles** Item-based CF answers "you liked movie A, here are movies similar to A." It cannot capture that you _specifically_ liked A for the acting (not the genre) and route you toward other actor-driven dramas. It treats A as a monolithic point. User-based can, in principle, find a peer who also values acting over genre.

---

### Summary table

|Dimension|User-Based|Item-Based|
|---|---|---|
|Interpretability|High — peer group visible|High — "because you liked X"|
|Stability of similarities|Low — user tastes shift|High — items are static|
|Sparsity resilience|Moderate (38 co-ratings/pair)|Very low (0.017/pair in your data)|
|Precomputation|Hard — active user is new|Easier — item matrix is static|
|Cold start (new user)|Fails (no similarity vector)|Fails equally|
|Cold start (new item)|Fails equally|Fails (no ratings to compare)|
|Scale|O(U × I) per request|O(I² ) offline, O(k) online|
|Your ablation RMSE|**0.8280** (Pearson k=80)|Not comparable — SVD approximation used in server|

**Bottom line:** On your dataset, UserBased edges out pure ItemBased because the user axis is denser (1,000 users × avg 37 ratings = 37,000 cells) than the item axis (8,737 items × avg 37 ratings = same cells, but spread across exponentially more pairs). Both are outperformed by SVD (RMSE ~0.80) because latent factor models extract the global structure that sparsity hides from memory-based methods.



# **ITEM-BASED**
## Phase 1 — `fit()` : offline, runs once on the training data

**Step 1 — Index the ratings by item** ([models.py:914](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L914))

```python
self._item_ratings = {i: dict(trainset.ir[i]) for i in trainset.all_items()}
# result: { item_id: { user_id: rating, ... }, ... }
```

For every item, store which users rated it and what they gave.

**Step 2 — Compute every user's mean** ([models.py:915-917](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L915-L917))

```python
self._user_means = { u: mean of all ratings by u }
```

Needed later to remove user scale bias when computing similarity.

**Step 3 — Compute every item's mean** ([models.py:919-921](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L919-L921))

```python
self._item_means = { i: mean of all ratings for i }
```

Needed later as the baseline in the prediction formula.

No similarity matrix is built here. Similarities are computed on demand.

---

## Phase 2 — `_item_sim(i, j)` : called at prediction time, per item pair

**Step 4 — Find co-raters** ([models.py:928-930](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L928-L930))

```python
common = set(ri_d) & set(rj_d)
if len(common) < 3: return 0.0    # min_support guard
```

Users who rated both item i and item j. If fewer than 3, return 0 immediately.

**Step 5 — Remove user bias (adjusted cosine)** ([models.py:932-934](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L932-L934))

```python
ri = [ri_d[u] - self._user_means[u] for u in common]
rj = [rj_d[u] - self._user_means[u] for u in common]
```

For each co-rater u, subtract their mean from both ratings. This removes the "lenient/harsh rater" effect before measuring how similar the items are.

**Step 6 — Compute cosine on centred vectors** ([models.py:946-947](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L946-L947))

```python
denom = norm(ri) * norm(rj)
return dot(ri, rj) / denom
```

Standard cosine similarity, but on the already-centred vectors from step 5. Result is in [−1, +1].

---

## Phase 3 — `estimate(u, i)` : called once per (user, target item) pair

**Step 7 — Gather user's rated history** ([models.py:953](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L953))

```python
user_history = [(j, r) for j, r in trainset.ur[u] if j != i]
```

All movies this user has rated, excluding the target item i itself.

**Step 8 — Compute similarity between target item i and every item in history** ([models.py:957](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L957))

```python
sims = [(j, r, self._item_sim(i, j)) for j, r in user_history]
```

Calls `_item_sim` for each pair. This is the expensive step — O(history length × co-rater lookup).

**Step 9 — Keep top-k by absolute similarity, then discard negatives** ([models.py:958-959](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L958-L959))

```python
top_k = sorted(sims, key=lambda x: abs(x[2]), reverse=True)[:k]
pos   = [(j, r, s) for j, r, s in top_k if s > 0]
```

Sort by `|sim|` to keep the most relevant neighbours. Then discard any with sim ≤ 0.

**Step 10 — Predict using KNNWithMeans formula** ([models.py:964-966](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L964-L966))

```python
mu_i  = self._item_means[i]
numer = sum(s * (r - self._item_means[j]) for j, r, s in pos)
denom = sum(abs(s) for _, _, s in pos)
return clip(mu_i + numer / denom, 0.5, 5.0)
```

Start from item i's mean. For each neighbour j: take the user's rating of j, subtract j's mean (the deviation), weight it by sim(i,j), sum it all up. Add that correction to μ_i.

---

## The full pipeline on one example

Say you want to predict your rating for **Alien**, and you have rated Predator (4.2) and Avatar (3.5):

```
Step 4: co-raters(Alien, Predator) = {u1, u2, u5, ...}  → 12 common, ≥ 3 ✓
Step 5: subtract each user's mean from their Alien/Predator ratings
Step 6: cosine → sim(Alien, Predator) = 0.71
        cosine → sim(Alien, Avatar)   = 0.43

Step 9: top-k = [Predator(0.71), Avatar(0.43)], both positive ✓

Step 10:
  μ_Alien = 4.1
  numer = 0.71 × (4.2 − μ_Predator) + 0.43 × (3.5 − μ_Avatar)
        = 0.71 × (4.2 − 3.6)  + 0.43 × (3.5 − 4.0)
        = 0.71 × 0.6           + 0.43 × (−0.5)
        = 0.426                − 0.215  = 0.211
  denom = 0.71 + 0.43 = 1.14
  r̂ = 4.1 + 0.211/1.14 = 4.1 + 0.185 = 4.29
```

You rated Predator above its average → push prediction up. You rated Avatar below its average → push prediction down. Net effect: slightly above Alien's mean.





# **USER-BASED :** 
# Phase 1 — Offline, runs once at server startup

**Step 1 — Build the ratings pivot matrix** ([server.py:179](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L179))

```python
_R_df = df_r.pivot(index=userId, columns=movieId, values=rating)
# shape: (1000 users × 8737 items), NaN where not rated
```

**Step 2 — Compute every training user's mean** ([server.py:183](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L183))

```python
_user_means_arr = _R_df.mean(axis=1)   # shape: (1000,)
```

**Step 3 — Mean-centre every training user's vector, fill missing with 0** ([server.py:189-191](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L189-L191))

```python
_R_centered = _R_df.sub(_R_df.mean(axis=1), axis=0).fillna(0.0)
_R_norm     = _R_centered / norm(_R_centered, axis=1)   # unit vectors
```

Each user row has their mean subtracted, then unrated items set to 0 ("no evidence"), then normalised to unit length. This is the Pearson similarity precomputation — every user vector is ready to be dot-producted.

**Step 4 — Keep the raw matrix too** ([server.py:194](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L194))

```python
_R_raw = _R_df.values   # NaN preserved — used in the prediction formula
```

---

## Phase 2 — Online, runs per request

**Step 5 — Build the active user's rating vector** ([server.py:1058-1063](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1058-L1063))

```python
new_vec = np.zeros(8737)
for mid, r in user_ratings.items():
    new_vec[idx] = r
    rated_mask[idx] = True
```

The active user arrives with ratings from this session. Slot them into an 8,737-dimensional vector.

**Step 6 — Compute active user's mean, centre, normalise** ([server.py:1065-1072](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1065-L1072))

```python
new_mean       = mean of rated positions
new_centered   = where(rated, new_vec - new_mean, 0.0)
new_normalized = new_centered / norm(new_centered)
```

Exactly the same transformation applied to training users in step 3. Unrated positions stay 0. This is the key — both the training users and the active user are in the same representation before computing similarity.

**Step 7 — Compute similarity to all 1,000 training users in one operation** ([server.py:1073](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1073))

```python
sims = _R_norm @ new_normalized   # (1000,) dot products at once
```

Because both sides are already unit-normalised and mean-centred, this dot product is exactly Pearson correlation for every training user simultaneously.

**Step 8 — Select top-k, discard negatives** ([server.py:1074-1086](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1074-L1086))

```python
top_idx  = argsort(sims)[::-1][:k]       # top-80 indices
pos_mask = top_sims > 0                   # drop negatives
pos_idx  = top_idx[pos_mask]
pos_sims = top_sims[pos_mask]
```

**Step 9 — Fetch those neighbours' raw ratings and means** ([server.py:1087-1092](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1087-L1092))

```python
pos_ratings  = _R_raw[pos_idx]            # (n_pos × 8737), NaN for unrated
pos_means    = _user_means_arr[pos_idx]   # (n_pos,)
centered_nbr = pos_ratings - pos_means    # subtract each neighbour's mean
centered_nbr = where(nan, 0.0, centered_nbr)   # NaN → 0 ("no evidence")
rated_by_nbr = (~isnan(pos_ratings)).float()   # 1 if rated, 0 if not
```

**Step 10 — Predict all 8,737 items at once** ([server.py:1093-1096](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1093-L1096))

```python
w          = pos_sims[:, newaxis]                      # weights column vector
numer      = (w * centered_nbr).sum(axis=0)            # (8737,)
denom      = (w * rated_by_nbr).sum(axis=0) + 1e-8    # (8737,)
all_scores = clip(new_mean + numer / denom, 0.5, 5.0)
```

Formula applied to all items simultaneously:

```
r̂(u,i) = μ_u  +  Σ sim(u,v) · (r_{v,i} − μ_v) / Σ sim(u,v)
```

---

## The same example, user-based style

Say you rated Predator (4.2) and Avatar (3.5). Your mean = 3.85.

Two training users surface as top neighbours:

- User 42: sim = 0.71, mean = 3.6, rated Alien = 4.5
- User 17: sim = 0.43, mean = 4.1, rated Alien = 4.0

```
Step 6: your centred vector: Predator = 4.2−3.85 = +0.35
                              Avatar  = 3.5−3.85 = −0.35

Step 7: sims = _R_norm @ your_normalised_vector → [0.71, 0.43, ...]

Step 9: centered_nbr for Alien:
         user42: 4.5 − 3.6 = +0.9
         user17: 4.0 − 4.1 = −0.1

Step 10:
  numer = 0.71 × 0.9  +  0.43 × (−0.1) = 0.639 − 0.043 = 0.596
  denom = 0.71 + 0.43 = 1.14
  r̂ = 3.85 + 0.596/1.14 = 3.85 + 0.52 = 4.37
```

User 42 loved Alien more than their average → strong push up. User 17 slightly disappointed → small push down. Net result: you'd likely rate Alien above your own average.

---

## Side-by-side comparison

| Step                    | Item-Based                                     | User-Based                                             |
| ----------------------- | ---------------------------------------------- | ------------------------------------------------------ |
| Offline prep            | Index ratings by item, compute user/item means | Build normalised user matrix `_R_norm`                 |
| Similarity axis         | Between **items** (over shared raters)         | Between **users** (over shared items)                  |
| Similarity computed     | On demand per item pair                        | All 1,000 at once via matrix multiply                  |
| Centring for similarity | Subtract **user** means (adjusted cosine)      | Subtract **user** means (Pearson)                      |
| Centring for prediction | Subtract **item** means (KNNWithMeans)         | Subtract **neighbour** means, add **active user** mean |
| Output                  | One prediction per call                        | All 8,737 predictions in one pass                      |




# **CONTENT-BASED :** 
## Phase 1 — Feature construction, runs once at import time

**Step 1 — Load the genome tag file** ([models.py:44](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L44))

The MovieLens genome is a matrix of 8,737 movies × 1,128 tags (e.g. "suspenseful", "visually stunning", "based on a book"), where each cell is a relevance score between 0 and 1.

```
             tag_1    tag_2    tag_3  ...  tag_1128
Toy Story    0.025    0.720    0.031  ...   0.041
Jumanji      0.041    0.310    0.055  ...   0.028
...
```

**Step 2 — Compress with TruncatedSVD** ([models.py:46](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L46))

```python
reduced = TruncatedSVD(n_components=150).fit_transform(df_pivot.values)
# 8737 × 1128  →  8737 × 150
```

1,128 tags compress into 150 latent dimensions. Each dimension captures a concept — one might represent "dark psychological thriller", another "family animation". Movies absent from the genome file are mean-imputed.

**Step 3 — Append 4 item quality stats** ([models.py:1248-1256](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L1248-L1256))

This is the `ContentBasedWithStats` addition on top of the base genome features:

```python
'item_bayesian_mean': (n·μ_i + 25·μ_global) / (n + 25)   # shrunk quality score
'item_log_count':     log(1 + n_ratings)                   # popularity
'item_std':           std of all ratings                    # divisiveness
'item_high_ratio':    fraction of ratings ≥ 4              # "loved it" rate
```

All four are MinMax-scaled to [0, 1] to match the genome feature range. Final feature vector per movie: **154 dimensions** (150 genome + 4 stats).

---

## Phase 2 — `fit()` : offline, runs once per user

**Step 4 — For each user independently, gather their rated items** ([models.py:1095-1100](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L1095-L1100))

```python
X = feature_matrix[items_user_rated]   # shape: (n_rated × 154)
y = user's ratings for those items     # shape: (n_rated,)
```

**Step 5 — Fit a Ridge regressor per user** ([models.py:1143-1147](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L1143-L1147))

```python
alphas = [5, 10, 20, 25, 50, 100, 200]
reg = RidgeCV(alphas=alphas).fit(X, y)
self.user_profile[u] = reg
```
		
Ridge solves:

```
min  ||X·w − y||²  +  α·||w||²
  w
```

The result is a weight vector `w_u` of 154 numbers — one per feature dimension. It encodes _this specific user's taste profile_: which genome dimensions they respond to positively, which stats they care about. A user who loves dark psychological thrillers gets large positive weights on those genome components.

`RidgeCV` automatically selects the best `α` from the grid via leave-one-out cross-validation within the user's own ratings. Users with few ratings get a larger α (stronger regularisation → simpler model → less overfitting). This is done independently for each of the 1,000 users.

---

## Phase 3 — `estimate(u, i)` : online, per (user, item) pair

**Step 6 — Look up the item's feature vector** ([models.py:1193](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L1193))

```python
x_i = content_features.loc[item_id].values   # shape: (154,)
```

**Step 7 — Dot product with user's weight vector** ([models.py:1201](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L1201))

```python
score = regressor.predict([x_i])[0]
# = w_u · x_i  (+ intercept)
```

That is the predicted rating. Clip to [0.5, 5.0].

---

## The full pipeline on one example

Say user u loves sci-fi (high relevance on genome dimension 12 = "space opera") and always rates popular films higher. Their Ridge fit produced:

```
w_u[genome_dim_12]       = +0.8   (strong positive on space opera)
w_u[item_bayesian_mean]  = +1.2   (rewards quality)
w_u[item_log_count]      = +0.3   (mild popularity boost)
... (other 151 weights near 0)
```

For Alien (x_i):

```
genome_dim_12        = 0.72   (high space opera relevance)
item_bayesian_mean   = 0.85   (scaled, high quality)
item_log_count       = 0.71   (many ratings)

score = 0.8×0.72 + 1.2×0.85 + 0.3×0.71 + intercept
      = 0.576   + 1.02    + 0.213   + ~1.5
      ≈ 3.3  →  clipped to [0.5, 5] → 3.3
```

The key insight: **the prediction is entirely based on the movie's content**, not on who else rated it. No user-user or item-item comparison. You could predict a rating for a brand new movie with zero ratings, as long as it has genome tags.

---

## Side-by-side: all three methods

||User-Based|Item-Based|Content-Based|
|---|---|---|---|
|What it uses|Other users' ratings|Your ratings on similar items|Movie features (tags, stats)|
|Trained on|Rating matrix|Rating matrix|Rating matrix + genome|
|Model per user|None (computed live)|None (computed live)|Fitted Ridge regressor|
|Cold start — new item|Fails|Fails|**Works** (just needs genome tags)|
|Cold start — new user|Fails|Fails|**Works** (fit on first few ratings)|
|Sparsity sensitivity|Moderate|High|**None** — no co-ratings needed|
|Explainability|Peer group + ratings|Similar movies|Feature weights per dimension|
|Your RMSE (CV)|0.8280|Worse (sparsity)|~0.84–0.86|





# **LATENT-FACTORS :**
## Phase 1 — Training, runs once at server startup

**Step 1 — Train Surprise SVD on all 381,181 ratings** ([server.py:302-306](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L302-L306))

```python
_svd_model = SurpriseSVD(n_factors=100, n_epochs=30, lr_all=0.005, reg_all=0.02)
_svd_model.fit(trainset)
```

SVD factorises the rating matrix by minimising this loss via SGD ([models.py:982](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/models.py#L982)):

```
L = Σ (r_{u,i} − μ − b_u − b_i − pᵤ · qᵢ)²  +  λ(||pᵤ||² + ||qᵢ||² + b_u² + b_i²)
```

Four things are learned simultaneously:

- `μ` — global mean rating (scalar)
- `b_u` — user bias (does this user rate higher/lower than average?)
- `b_i` — item bias (is this item rated higher/lower than average?)
- `pᵤ` — user latent vector (100 dims, what kind of films does this user like?)
- `qᵢ` — item latent vector (100 dims, what kind of film is this?)

After 30 epochs of SGD over 381k ratings, every training user and every item has a position in the 100-dimensional latent space.

**Step 2 — Extract and store only the item side** ([server.py:312-316](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L312-L316))

```python
_global_mean  = trainset.global_mean         # scalar
_item_biases  = [_svd_model.bi[i] ...]       # (8737,)
_item_factors = [_svd_model.qi[i] ...]       # (8737 × 100)
```

The training user vectors `pᵤ` are discarded. The active user (who arrives at request time) was never in the training set — their `pᵤ` does not exist yet.

---

## Phase 2 — Online, runs per request

**Step 3 — Remove the bias signal from the active user's ratings** ([server.py:1196-1199](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1196-L1199))

```python
Q_rated    = _item_factors[rated_idx]   # item vectors for rated movies (n_rated × 100)
b_rated    = _item_biases[rated_idx]    # item biases for rated movies  (n_rated,)
y_centered = y_vals - _global_mean - b_rated
```

Before learning the user's latent vector, strip out everything that is not the user's personal taste:

- subtract `μ` (global mean) — everyone gives around 3.5 on average
- subtract `b_i` for each rated movie — The Godfather gets rated high by everyone, that's not your personal signal

What remains in `y_centered` is the **residual**: the part of your rating that is genuinely _your_ preference, beyond what any user would give that film.

**Step 4 — Fold the active user into the latent space via Ridge** ([server.py:1200-1203](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1200-L1203))

```python
lam = max(0.02, 1.0 / n_rated)
reg = Ridge(alpha=lam, fit_intercept=True)
reg.fit(Q_rated, y_centered)
p_u = reg.coef_     # (100,)  ← the user's latent vector
b_u = reg.intercept_
```

This solves:

```
min  ||Q_rated · pᵤ − y_centered||²  +  λ·||pᵤ||²
 pᵤ
```

In words: find the 100-dimensional vector `pᵤ` that best reconstructs your residual ratings from the pre-trained item vectors. This places you inside the same latent space as the training users, without ever retraining the model.

The regularisation `λ = 1/n_rated` automatically adapts: with 5 ratings you get heavy regularisation (shrink toward zero = don't trust too much), with 200 ratings you get weak regularisation (trust the signal).

**Step 5 — Score all 8,737 items in one matrix operation** ([server.py:1205-1208](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/server.py#L1205-L1208))

```python
all_scores = clip(
    _global_mean + b_u + _item_biases + _item_factors @ p_u,
    0.5, 5.0
)
```

The full prediction formula:

```
r̂(u,i) = μ  +  b_u  +  b_i  +  pᵤ · qᵢ
```

- `μ` — start from the global baseline
- `b_u` — shift up/down based on your rating generosity
- `b_i` — shift up/down based on the movie's general quality
- `pᵤ · qᵢ` — the dot product between your taste vector and the movie's latent representation

This is O(n_items × k) = 8,737 × 100 = 873,700 multiplications — computed as a single matrix multiply, imperceptible at serving time.

---

## Concrete example

Say you rated Alien (4.5), Predator (4.0), and Toy Story (2.0). After step 3:

```
y_centered[Alien]    = 4.5 − 3.5 − b_Alien    ≈ 4.5 − 3.5 − 0.6   = 0.4
y_centered[Predator] = 4.0 − 3.5 − b_Predator ≈ 4.0 − 3.5 − 0.2   = 0.3
y_centered[Toy Story]= 2.0 − 3.5 − b_ToyStory ≈ 2.0 − 3.5 − 0.5   = −2.0
```

Ridge finds `pᵤ` such that `Q_rated @ pᵤ ≈ [0.4, 0.3, −2.0]`. The resulting `pᵤ` will have large positive values on the "sci-fi/action" latent dimensions and large negative values on the "family/animation" dimensions.

For The Thing (unseen, qi heavily loaded on "sci-fi/horror"):

```
r̂ = 3.5 + b_u + b_TheThing + pᵤ · q_TheThing
   ≈ 3.5 + 0.1 + 0.3        + 0.85              ≈ 4.75
```

For Shrek (qi loaded on "family/animation"):

```
r̂ = 3.5 + 0.1 + 0.4 + (−0.6) ≈ 3.4
```

---

## Side-by-side: all four methods

|User-Based|Item-Based|Content-Based|**SVD**|
|---|---|---|---|---|
|Core idea|Similar users|Similar items|Movie features|Latent dimensions|
|Offline artefact|Normalised user matrix|Item factor matrix|Per-user Ridge model|`μ, b_i, qᵢ` for all items|
|Online step|Matrix multiply (sims)|Item-sim per pair|One dot product|Ridge fold-in + matrix multiply|
|New user|Recomputes live|Recomputes live|Fits Ridge live|**Folds in via Ridge**|
|New item cold start|Fails|Fails|Works (genome tags)|Fails (no `qᵢ`)|
|Sparsity resilience|Moderate|Low|None needed|**High** (global signal)|
|Your CV RMSE|0.8280|worse|~0.84–0.86|**~0.80**|




## TruncatedSVD sur la matrice de tags (genome) → toujours content-based

```python
# models.py — _genome_svd_features()
df_pivot = load_genome()          # 8737 films × 1128 tags (scores de pertinence)
reduced  = TruncatedSVD(n_components=150).fit_transform(df_pivot.values)
# résultat : 8737 × 150
```

La matrice d'entrée ici est une matrice de **contenu** — des scores de pertinence de tags assignés par des humains ("suspenseful" = 0.72, "visually stunning" = 0.65...). Aucun rating utilisateur n'est impliqué.

Le TruncatedSVD est ici un simple **outil de compression**. Il réduit 1128 dimensions redondantes en 150 dimensions latentes de contenu ("space opera", "dark psychological thriller"...). Ce sont des dimensions de **description de film**, pas de comportement utilisateur.

→ **C'est toujours du content-based.** Le SVD n'apporte pas de signal collaboratif.

---

## TruncatedSVD sur la matrice de ratings → là oui, latent factors CF

```python
# server.py — ItemBased dans le serveur
R = pivot(users × items, values=ratings)   # matrice de RATINGS
reduced = TruncatedSVD(n_components=k).fit_transform(R.T)
# résultat : vecteurs latents des items basés sur les comportements utilisateurs
```

Ici la matrice d'entrée est une matrice de **ratings**. Les dimensions latentes capturent des patterns de comportement collectif — "les gens qui aiment X aiment aussi Y". C'est du CF.

→ **C'est du latent factor collaboratif.**

---

## La règle

|Matrice d'entrée du SVD|Signal|Résultat|
|---|---|---|
|Genome tags (contenu)|Ce que les humains disent du film|Content-based latent|
|Ratings utilisateurs|Ce que les utilisateurs font|CF latent factor|
|Les deux combinés|Les deux|**Hybride**|

---

## Ton vrai hybride c'est HybridSVD

```python
# Composante CF : SVD entraîné sur les ratings
r̂_svd  = μ + b_u + b_i + pᵤ · qᵢ

# Composante content : Ridge sur genome tags
r̂_cb   = w_u · x_i

# Blend final
r̂      = α · r̂_svd + (1−α) · r̂_cb
```

C'est lui le vrai hybride — il combine explicitement un signal CF (ratings) et un signal content (genome). ContentBasedWithStats avec SVD sur les tags reste du content-based, juste avec une meilleure représentation des features.

okay SVD c'est quoi exactement ?

## SVD au sens mathématique

SVD = **Singular Value Decomposition**. C'est une décomposition de n'importe quelle matrice en trois matrices :

```
M  =  U · Σ · Vᵀ
```

Prenons ta matrice de ratings (1000 users × 8737 films) :

```
         Alien  Godfather  Toy Story  ...
user_1  [  4.5      NaN       2.0    ]
user_2  [  NaN      5.0       4.0    ]   =  U · Σ · Vᵀ
user_3  [  3.0      4.5       NaN    ]
...
```

Les trois matrices :

```
U  (1000 × k)   — chaque ligne = un vecteur utilisateur dans l'espace latent
Σ  (k × k)      — matrice diagonale, valeurs singulières (importance de chaque dimension)
Vᵀ (k × 8737)  — chaque colonne = un vecteur film dans l'espace latent
```

**TruncatedSVD** garde uniquement les `k` dimensions les plus importantes (celles avec les plus grandes valeurs singulières). Les autres dimensions captent du bruit et sont jetées.

```
M ≈ U_k · Σ_k · Vᵀ_k    (approximation de rang k)
```

---

## Pourquoi ça fonctionne pour la recommandation

Les dimensions latentes capturent des **patterns cachés** dans les comportements collectifs. Par exemple, si beaucoup d'utilisateurs qui aiment Alien aiment aussi The Thing et Predator, le SVD va créer une dimension latente "sci-fi/action" et mettre ces trois films proches dans cet espace, sans que personne n'ait explicitement défini ce concept.

```
dimension 1 ≈ "sci-fi/action"     : Alien(+0.8), The Thing(+0.7), Toy Story(-0.4)
dimension 2 ≈ "family/animation"  : Toy Story(+0.9), Shrek(+0.8), Alien(-0.3)
dimension 3 ≈ "crime/drama"       : Godfather(+0.9), Goodfellas(+0.8), ...
```

---

## ATTENTION : le SVD de Surprise n'est pas le vrai SVD

C'est là où c'est trompeur. Ce que Surprise appelle "SVD" est en réalité la **factorisation matricielle biaisée de Simon Funk** (Netflix Prize, 2006). Le nom est historiquement mal choisi.

Le vrai SVD résout la décomposition analytiquement. Le SVD de Surprise minimise une loss via SGD :

```
L = Σ (r_{u,i} − μ − b_u − b_i − pᵤ·qᵢ)²  +  λ(||pᵤ||² + ||qᵢ||² + b_u² + b_i²)
```

Pas de décomposition exacte. Il **apprend** `pᵤ` et `qᵢ` par descente de gradient, en ignorant les NaN (ratings manquants). Le vrai SVD ne peut pas gérer les NaN directement.

---

## Les deux dans ton code

|Utilisation|Quoi exactement|Sur quoi|
|---|---|---|
|`TruncatedSVD` (sklearn)|Vrai SVD tronqué|Matrice genome tags (contenu)|
|`SVD` (Surprise)|Factorisation matricielle biaisée par SGD|Matrice de ratings (CF)|

---

## Intuition géométrique

Imagine chaque film comme un point dans un espace à 8737 dimensions (une par utilisateur). C'est inexploitable. SVD trouve les **k directions** qui expliquent le plus de variance dans ces points, et projette tout le monde dans cet espace réduit à k dimensions.

```
Avant SVD : film = vecteur de 8737 ratings (dont 95% de NaN)
Après SVD  : film = vecteur de 100 nombres denses  ← qᵢ
```

Deux films proches dans cet espace de 100 dimensions → les mêmes utilisateurs les ont aimés de la même façon → ils sont "similaires" au sens collaboratif.

**Memory-based (UserBased) :**

- Stocke toute la matrice de ratings en mémoire
- Aucune phase de training — il calcule les similarités Pearson **au moment de la requête**
- Inference lente : O(n_users × n_items) à chaque appel
- Pas de compression — si t'as 100k users, tu gardes 100k vecteurs

**Model-based (SVD) :**

- Compresse la matrice R (n_users × n_items) en deux petites matrices P (n_users × k) et Q (n_items × k)
- Phase de training coûteuse (SGD sur tous les ratings), mais **une seule fois**
- Inference ultra-rapide : juste un produit scalaire `pᵤ · qᵢ` — O(k)
- k = 100 au lieu de 100k colonnes → généralise mieux


**Cosine :**

$$\text{sim}(u,v) = \frac{\sum r_{ui} \cdot r_{vi}}{\sqrt{\sum r_{ui}^2} \cdot \sqrt{\sum r_{vi}^2}}$$

Raw dot product divided by the product of norms. No centering.

---

**Pearson :**

$$\text{sim}(u,v) = \frac{\sum (r_{ui} - \bar{r}_u)(r_{vi} - \bar{r}_v)}{\sqrt{\sum (r_{ui} - \bar{r}_u)^2} \cdot \sqrt{\sum (r_{vi} - \bar{r}_v)^2}}$$

Exact same formula but on **mean-centred** vectors — subtract each user's average rating first.





**Information Gain (entropie)** $$IG(feature) = H(\text{parent}) - \sum \frac{|nœud_k|}{|parent|} \cdot H(nœud_k)$$

Combien d'entropie je **gagne** en splitant sur cette feature ? Plus le gain est élevé → feature plus importante.

**Problème :** favorise les features avec beaucoup de valeurs distinctes (ex: un ID unique aurait IG maximal mais serait inutile).

→ Corrigé par le **Gain Ratio** (C4.5) qui pénalise les features trop fragmentées.

---

**Gini Importance** Même logique mais avec Gini à la place de l'entropie. C'est ce que `feature_importances_` retourne dans un Random Forest sklearn — la **réduction moyenne de Gini** apportée par cette feature sur tous les arbres.

---

**Chi-squared vs IG vs Gini en feature selection :**

|Métrique|Type de feature|Remarque|
|---|---|---|
|Chi-squared|Catégorielle vs target catégorielle|Test statistique|
|Information Gain|Catégorielle ou continue|Sensible au nombre de valeurs|
|Gini Importance|Continue ou catégorielle|Utilisé dans Random Forest|

En pratique pour un RecSys : ces métriques s'appliquent peu — on préfère Ridge (coefficient magnitude) ou TruncatedSVD pour sélectionner/compresser les features comme les 1128 genome tags.
---

**Chi-squared (χ²) — feature selection** $$\chi^2 = \sum \frac{(O - E)^2}{E}$$

Teste si une feature catégorielle est **indépendante** de la target. O = fréquence observée, E = fréquence attendue si indépendants. Plus χ² est grand → plus la feature est dépendante de la target → plus elle est utile.

Utilisé pour **filtrer les features** avant d'entraîner : tu gardes les k features avec le χ² le plus élevé.



**Deviance** — c'est la généralisation de l'entropie pour les modèles probabilistes.

$$\text{Dev} = -2 \sum \log P(y_i | \hat{y}_i)$$

C'est la **log-vraisemblance négative multipliée par 2**. Mesure à quel point le modèle est surpris par les vraies valeurs — si le modèle prédit bien, la vraisemblance est haute, la deviance est basse.

**Dans les arbres de décision :** la deviance est utilisée comme critère de split à la place de Gini/entropie quand on travaille avec des **modèles probabilistes** (régression logistique dans chaque feuille par exemple).

**Différence avec l'entropie :**

- Entropie = mesure de l'impureté basée sur les fréquences observées dans le nœud
- Deviance = mesure basée sur la vraisemblance du modèle → plus générale, applicable aux variables continues aussi

**En pratique :** tu le verras surtout dans les **GLM** (Generalized Linear Models) et les **arbres de régression** où l'entropie classique ne s'applique pas directement car la target est continue.

Dans ton projet RecSys spécifiquement, ce n'est pas utilisé — Ridge minimise les moindres carrés, pas la deviance. Mais c'est un concept lié à la **log-loss** qu'on utilise en classification.


**Ratings binaires (aimé / pas aimé — 0 ou 1)**

**Gini** est idéal ici.

$$\text{Gini} = 1 - (p_{\text{aimé}}^2 + p_{\text{pas aimé}}^2)$$

Exemple : un nœud avec 80% aimé / 20% pas aimé : $$\text{Gini} = 1 - (0.8^2 + 0.2^2) = 1 - 0.68 = 0.32$$

Simple, rapide, parfait pour deux classes.

---

**Ratings discrets petit nombre (1 à 5 étoiles)**

**Entropie** est meilleure car elle gère bien **plusieurs classes**.

$$H = -(p_1\log p_1 + p_2\log p_2 + p_3\log p_3 + p_4\log p_4 + p_5\log p_5)$$

Si tous les ratings sont concentrés sur 4★ → entropie basse → nœud quasi pur. Si ratings répartis équitablement sur 1★ à 5★ → entropie maximale → nœud très impur.

Entropie est plus sensible que Gini quand les classes sont nombreuses — elle pénalise davantage les distributions uniformes.

---

**Ratings continus / grand nombre de valeurs (ex: scores 0 à 100)**

**Deviance** (ou variance pour la régression) devient la métrique adaptée car Gini et entropie supposent des classes discrètes.

$$\text{Dev} = -2\sum \log P(y_i | \hat{y}_i)$$

Ou plus simplement en régression, on minimise la **variance intra-nœud** : $$\text{Variance} = \frac{1}{n}\sum(y_i - \bar{y})^2$$





**Information Gain (entropie)** $$IG(feature) = H(\text{parent}) - \sum \frac{|nœud_k|}{|parent|} \cdot H(nœud_k)$$

Combien d'entropie je **gagne** en splitant sur cette feature ? Plus le gain est élevé → feature plus importante.

**Problème :** favorise les features avec beaucoup de valeurs distinctes (ex: un ID unique aurait IG maximal mais serait inutile).

→ Corrigé par le **Gain Ratio** (C4.5) qui pénalise les features trop fragmentées.

---

**Gini Importance** Même logique mais avec Gini à la place de l'entropie. C'est ce que `feature_importances_` retourne dans un Random Forest sklearn — la **réduction moyenne de Gini** apportée par cette feature sur tous les arbres.




On veut apprendre **w** tel que **w · x_i ≈ r_i**. Sans régularisation, OLS minimise :

$$\mathcal{L} = \sum_{i} (r_i - \mathbf{w}^\top \mathbf{x}_i)^2$$

Problème : avec 1128 features et peu de ratings, OLS **overfitte** — il trouve des poids énormes qui collent parfaitement au train mais généralisent mal.

---

**Ridge ajoute une pénalité**

$$\mathcal{L}_{ridge} = \underbrace{\sum_{i} (r_i - \mathbf{w}^\top \mathbf{x}_i)^2}_{\text{fit}} + \underbrace{\alpha |\mathbf{w}|^2}_{\text{pénalité}}$$

**Chaque terme :**

- $r_i$ — le vrai rating observé
- $\mathbf{w}^\top \mathbf{x}_i$ — la prédiction du modèle
- $(r_i - \mathbf{w}^\top \mathbf{x}_i)^2$ — l'erreur au carré sur ce rating
- $|\mathbf{w}|^2 = \sum_j w_j^2$ — somme des poids au carré (norme L2)
- $\alpha$ — force de régularisation. Grand α → poids très petits → underfitting. Petit α → poids libres → overfitting

---

**La solution analytique exacte**

$$\mathbf{w}^* = (X^\top X + \alpha I)^{-1} X^\top \mathbf{r}$$

- $X$ — matrice des features (items × 1128)
- $X^\top X$ — matrice de covariance des features
- $\alpha I$ — on ajoute α sur la diagonale → rend la matrice **toujours inversible**
- $X^\top \mathbf{r}$ — corrélation entre features et ratings

---

**L'effet concret**

Ridge **shrink** tous les poids vers zéro proportionnellement. Les features peu informatives finissent avec des poids quasi nuls — Ridge fait une soft feature selection automatiquement.

---

**Dans ton projet**

α=20 est optimal. Trop petit (α=1) → RMSE=0.8079 (overfit). Trop grand (α=200) → RMSE=0.7664 (underfit). Le sweet spot α=20 donne RMSE=0.7353.