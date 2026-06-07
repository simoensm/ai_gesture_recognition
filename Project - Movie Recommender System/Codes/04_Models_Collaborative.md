---
tags:
  - recommender-systems
  - code
  - python
  - collaborative-filtering
  - MLSMM2156
---
# Collaborative Filtering Models

**File:** `recsys/models.py` — `UserBased` (line 790) and `ItemBased` (line 892)

Both extend Surprise's `AlgoBase`. For the theoretical background, see [[Workflow/08_Results_and_Decision_Making]].

---

## `UserBased` — User-Based KNN

### The Core Formula

$$\hat{r}_{u,i} = \mu_u + \frac{\sum_{v \in N_k(u,i)} \text{sim}(u,v)\,(r_{v,i} - \mu_v)}{\sum_{v \in N_k(u,i)} \text{sim}(u,v)}$$

Plain English: "Predict user u's rating for item i as user u's average rating, plus the weighted average deviation of similar users' ratings for item i from *their* averages."

Mean-centering is key: a generous rater who gives 4–5 stars and a harsh rater who gives 2–3 stars can still be similar in *taste*. By subtracting each user's mean (μᵥ), we compare their preferences on a common scale.

---

### `fit(trainset)` — Building the Similarity Matrix

```python
def fit(self, trainset):
    # 1. Compute per-user means
    self._user_means = np.array([...])
    
    # 2. Build the rating matrix R: shape (n_users, n_items)
    R = np.full((m, n), np.nan)
    for u in range(m):
        for i, r in trainset.ur[u]:
            R[u, i] = r
    
    # 3. Compute pairwise user similarity
    self.sim = np.eye(m)   # diagonal = 1 (user is perfectly similar to themselves)
    for u in range(m):
        for v in range(u + 1, m):
            ...
            self.sim[u, v] = self.sim[v, u] = s   # symmetric
```

**Cost:** O(n_users²) in both time and memory. With ~1,000 users, this is a 1,000×1,000 matrix — manageable. At 100,000 users, this becomes infeasible.

### Similarity Metrics

**MSD (Mean Squared Difference):**
```python
msd = np.nansum((R[u] - R[v]) ** 2) / support
s = 1.0 / (msd + 1.0)
```
Penalises large rating differences. `np.nansum` ignores NaN (unrated) pairs. `support` = number of co-rated items. The +1 ensures similarity is in (0, 1] even when MSD=0.

**Jaccard:**
```python
s = np.sum(rated_u & rated_v) / np.sum(rated_u | rated_v)
```
Pure set overlap: how many items did both users rate, divided by how many either user rated. Ignores rating *values* entirely.

**Spearman:**
```python
from scipy.stats import rankdata
d_sq = np.sum((rankdata(ru) - rankdata(rv)) ** 2)
s = 1.0 - 6.0 * d_sq / (n * (n**2 - 1))
```
Converts ratings to ranks before computing correlation. Robust to outlier ratings; not available in Surprise's optimised C implementation, so it's in our custom Python class.

### `estimate(u, i)` — Making a Prediction

```python
def estimate(self, u, i):
    est = self._user_means[u]    # start from user's own mean
    
    # Get top-k most similar users who rated item i
    neighbors = [(v, self.sim[u, v], r_vi) for v, r_vi in self.trainset.ir[i] if v != u]
    top_k = heapq.nlargest(self.k, neighbors, key=lambda t: t[1])
    
    # Weighted average of deviations
    numer = sum(s * (r - self._user_means[v]) for v, s, r in top_k if s > 0)
    denom = sum(s for _, s, _ in top_k if s > 0)
    
    if actual_k >= self.min_k and denom > 0:
        est += numer / denom
    
    return float(np.clip(est, lo, hi))
```

`heapq.nlargest` is more efficient than `sorted()` when k << n — it finds the k largest elements in O(n log k) instead of O(n log n).

---

## `ItemBased` — Item-Based KNN

### The Core Formula

$$\hat{r}_{u,i} = \mu_i + \frac{\sum_{j \in N_k(i,u)} \text{sim}(i,j)\,(r_{u,j} - \mu_j)}{\sum_{j \in N_k(i,u)} |\text{sim}(i,j)|}$$

Plain English: "Predict u's rating for item i as item i's average, plus the weighted average deviation of similar items' ratings by user u from *their* averages."

---

### Key Design Difference vs `UserBased`

**`UserBased`** pre-computes the full n_users × n_users similarity matrix in `fit()`. Fast at `estimate()` time.

**`ItemBased`** computes item similarities **on demand** in `estimate()`, only between item i and the items user u has rated. This keeps memory linear in the number of ratings (not quadratic in items). But it's slower per prediction — each `estimate()` call iterates over the user's history.

```python
def estimate(self, u, i):
    user_history = [(j, r) for j, r in self.trainset.ur[u] if j != i]
    sims = [(j, r, self._item_sim(i, j)) for j, r in user_history]
    top_k = sorted(sims, key=lambda x: abs(x[2]), reverse=True)[:self.k]
    ...
```

### `_item_sim(i, j)` — On-Demand Similarity

```python
def _item_sim(self, i, j):
    common = set(ri_d) & set(rj_d)   # users who rated BOTH items
    if len(common) < self.min_support:
        return 0.0
    
    if self.similarity_method == 'adj_cosine':
        ri = [ri_d[u] - self._user_means[u] for u in common]   # mean-center per USER
        rj = [rj_d[u] - self._user_means[u] for u in common]
        # then cosine similarity on the centered vectors
```

**Adjusted Cosine** (Sarwar 2001 default): subtracts the *user* mean (not item mean) before computing cosine. This removes the rating-scale bias of different users when comparing items.

---

> [!WARNING]+ ❓ Assessor Question — Why does item-based CF outperform user-based CF on MovieLens?
> Item neighbourhoods are more stable than user neighbourhoods for two reasons: (1) Popular movies accumulate hundreds or thousands of ratings from many different users, giving reliable co-rating statistics. User taste profiles, on the other hand, are noisier and shift over time. (2) The adjusted-cosine formula already corrects for user-scale differences when computing item similarity, partially redundifying the Pearson correction that helped user-based CF. In this dataset, MSD beats Pearson for item-based CF — the inverse of the user-based result — because item vectors are already "scale-corrected" by the adjusted-cosine preprocessing.

> [!WARNING]+ ❓ Assessor Question — What is `min_support` and why does it matter?
> `min_support` is the minimum number of items that two users must have co-rated (or users that two items must share) for their similarity to be computed. Without this threshold, a similarity computed from a single co-rated item is statistically meaningless — two users who both rated only one movie are "100% similar" by any metric on that one data point. Setting `min_support=3` requires at least 3 co-rated items, ensuring similarity estimates are based on enough data to be reliable.

> [!WARNING]+ ❓ Assessor Question — Why is `np.clip(est, lo, hi)` needed?
> The weighted average formula can produce estimates outside the [0.5, 5.0] valid rating range. For example, if user u has mean 4.5 and all their similar users also rated item i at 5.0 (their maximum), the formula would predict 4.5 + large positive number, which could exceed 5.0. Clipping ensures predictions stay within the valid scale, preventing inflated RMSE from impossible-rating predictions.

---

**See also:** [[Workflow/08_Results_and_Decision_Making]] | [[Codes/03_Models_Baselines]] | [[Codes/05_Models_Latent_Factor]]
