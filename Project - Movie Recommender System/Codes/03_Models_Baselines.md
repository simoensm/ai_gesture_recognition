# Baseline Models

**File:** `recsys/models.py` — lines 748–783

Baselines exist to benchmark. Any useful model must beat all of them.

---

## The Surprise `AlgoBase` Interface

All models (including baselines) extend Surprise's `AlgoBase`. The contract is simple:

```python
class MyModel(AlgoBase):
    def fit(self, trainset):
        # Learn from training data
        AlgoBase.fit(self, trainset)   # always call super
        return self
    
    def estimate(self, u, i):
        # Return a predicted rating for user u, item i (Surprise inner IDs)
        return some_float
```

- `trainset`: a Surprise `Trainset` object — contains all ratings, user/item ID mappings, and rating scale
- `u`, `i`: **Surprise inner IDs**, not the raw userIds/movieIds. Use `trainset.to_raw_uid(u)` and `trainset.to_raw_iid(i)` to convert.

---

## `ModelBaseline1` — Constant Predictor

```python
class ModelBaseline1(AlgoBase):
    def estimate(self, u, i):
        return 2
```

Always returns rating 2. No training needed. This is the absolute floor — if your model doesn't beat this, something is very wrong.

---

## `ModelBaseline2` — Random Uniform

```python
class ModelBaseline2(AlgoBase):
    def fit(self, trainset):
        AlgoBase.fit(self, trainset)
        rd.seed(0)
    
    def estimate(self, u, i):
        return rd.uniform(self.trainset.rating_scale[0],
                          self.trainset.rating_scale[1])
```

Returns a random float between 0.5 and 5.0. `rd.seed(0)` in `fit()` ensures reproducibility: the same sequence of random numbers is generated on every run. `self.trainset.rating_scale` reads the scale from the loaded dataset (0.5, 5.0) rather than hardcoding it.

---

## `ModelBaseline3` — Global Mean

```python
class ModelBaseline3(AlgoBase):
    def fit(self, trainset):
        AlgoBase.fit(self, trainset)
        self.the_mean = np.mean([r for (_, _, r) in self.trainset.all_ratings()])
        return self
    
    def estimate(self, u, i):
        return self.the_mean
```

Computes the average rating across ALL training ratings and returns it for every prediction. This is the most important baseline: any personalised model must beat it. Expected RMSE ≈ 1.05.

`trainset.all_ratings()` is a Surprise generator that yields `(inner_user_id, inner_item_id, rating)` tuples for every rating in the training set.

---

## `ModelBaseline4` — Vanilla SVD

```python
class ModelBaseline4(SVD):
    def __init__(self):
        SVD.__init__(self, n_factors=100, random_state=1)
```

Wraps Surprise's SVD with default hyperparameters (λ=0.02, η=0.005) and 100 factors. This is the "out-of-the-box" SVD result, before any tuning. Expected RMSE ≈ 0.81.

It shows how much hyperparameter tuning matters: `ModelBaseline4` vs. `SVDModel` with tuned hyperparameters saves ~0.02 RMSE.

---

## `get_top_n(predictions, n)` — Utility

```python
def get_top_n(predictions, n):
    rd.seed(0)
    top_n = defaultdict(list)
    for uid, iid, true_r, est, _ in predictions:
        top_n[uid].append((iid, est))
    for uid, user_ratings in top_n.items():
        rd.shuffle(user_ratings)
        user_ratings.sort(key=lambda x: x[1], reverse=True)
        top_n[uid] = user_ratings[:n]
    return top_n
```

Given a list of `Prediction` objects (one per (user, item) pair), groups them by user and returns the top-n items per user sorted by predicted rating.

**Why `rd.shuffle` before sorting?** To break ties randomly rather than always picking the same item when two items have identical predicted ratings. This prevents the same tie-breaking pattern across all users, which would artificially reduce diversity metrics.

---

## `_compute_baselines(trainset, lambda_b=25)` — Bias Helper

```python
def _compute_baselines(trainset, lambda_b=25):
    mu = trainset.global_mean
    
    b_i = {}
    for i in trainset.all_items():
        ratings_i = [r for _, r in trainset.ir[i]]
        b_i[i] = sum(r - mu for r in ratings_i) / (lambda_b + len(ratings_i))
    
    b_u = {}
    for u in trainset.all_users():
        b_u[u] = sum(r - mu - b_i.get(i, 0.0) for i, r in trainset.ur[u]) / (lambda_b + len(trainset.ur[u]))
    
    return mu, b_u, b_i
```

Computes **Koren (2009) shrinkage biases**:
- `b_i`: item bias = how much this item deviates from the global mean (on average)
- `b_u`: user bias = how much this user deviates from the mean, after accounting for item biases

The `lambda_b = 25` term in the denominator is the **shrinkage parameter**: items/users with few ratings are pulled toward 0 (unbiased) to prevent extreme values from small samples.

Used by the `_bias` regressor variants in `ContentBased`.

---

> [!WARNING]+ ❓ Assessor Question — What does Surprise's `trainset.ur[u]` contain?
> `trainset.ur` is a dictionary mapping each inner user ID to a list of `(inner_item_id, rating)` tuples — all the items this user rated in the training set. Similarly, `trainset.ir[i]` maps an inner item ID to `(inner_user_id, rating)` tuples. These are the core data structures Surprise exposes for building custom algorithms.

> [!WARNING]+ ❓ Assessor Question — Why does ModelBaseline3 beat ModelBaseline1 on RMSE?
> Baseline1 always predicts 2, but the global mean is ~3.5. So Baseline1 systematically under-predicts by ~1.5 stars. Baseline3 predicts the global mean, so its errors are centred around zero — the average error is 0 (unbiased). RMSE rewards being unbiased: the mean predictor minimises RMSE among all constant predictors, since RMSE² = bias² + variance and the variance of a constant is 0.

---

**See also:** [[Codes/04_Models_Collaborative]] | [[Codes/05_Models_Latent_Factor]] | [[Workflow/01_Project_Overview]]
