---
tags:
  - recommender-systems
  - code
  - python
  - matrix-factorization
  - SVD
  - MLSMM2156
---
# Latent Factor Models

**File:** `recsys/models.py` — `SVDModel` (line 982), `SVDppModel` (line 1006), `NMFModel` (line 1027)

All three wrap Surprise's implementations with default hyperparameters set to their **tuned** values. For results, see [[Workflow/05_Hyperparameter_Tuning]].

---

## The Core Idea

Instead of finding similar users or items directly, latent factor models **compress** the entire rating matrix into low-dimensional user and item vectors. A user's taste is encoded as a k-dimensional vector (embedding), and an item's characteristics are encoded as another k-dimensional vector. The predicted rating is derived from these vectors.

This learns "latent" (hidden) dimensions that might correspond to real concepts like "action preference", "art-house vs. mainstream", etc. — but they're discovered automatically, not labelled.

---

## `SVDModel` — Biased Matrix Factorisation

```python
class SVDModel(SVD):
    def __init__(self, n_factors=100, n_epochs=30, lr_all=0.005, reg_all=0.02):
        SVD.__init__(self, n_factors=n_factors, n_epochs=n_epochs,
                     lr_all=lr_all, reg_all=reg_all, random_state=42)
```

### The Model

$$\hat{r}_{u,i} = \mu + b_u + b_i + \mathbf{p}_u^\top \mathbf{q}_i$$

- **μ**: global mean (~3.5)
- **b_u**: user bias — how much user u rates above/below the mean (e.g. harsh rater: b_u = -0.5)
- **b_i**: item bias — how much item i is rated above/below the mean (e.g. classic film: b_i = +0.8)
- **pᵤ · qᵢ**: dot product of user embedding (pᵤ ∈ ℝᵏ) and item embedding (qᵢ ∈ ℝᵏ) — captures personalised preference residual after biases

### The Loss Function

$$\mathcal{L} = \sum_{(u,i) \in \text{train}} (r_{u,i} - \hat{r}_{u,i})^2 + \lambda (\|\mathbf{p}_u\|^2 + \|\mathbf{q}_i\|^2 + b_u^2 + b_i^2)$$

The λ term is **L2 regularisation**: it penalises large weights, forcing the model to find simpler explanations. Without it, the model memorises training ratings but generalises poorly.

### Training: SGD (Stochastic Gradient Descent)

For each (u, i, r) in the training set, compute the error `e = r - r̂`, then:
```
b_u ← b_u + η (e - λ b_u)
b_i ← b_i + η (e - λ b_i)
pᵤ  ← pᵤ  + η (e·qᵢ - λ·pᵤ)
qᵢ  ← qᵢ  + η (e·pᵤ - λ·qᵢ)
```

Each update moves parameters in the direction that reduces the error for *this* rating.

### Tuned Hyperparameters

| Parameter | Default | Tuned | Effect of change |
|---|---|---|---|
| `n_factors` | 100 | 100 | More factors = more expressive but higher overfitting risk |
| `reg_all` (λ) | 0.02 | **0.05** | Higher = less overfitting. Default was too low → saved 0.017 RMSE |
| `lr_all` (η) | 0.005 | **0.010** | Higher = faster convergence. Too high = oscillation |
| `n_epochs` | 20 | 30 | More passes = better convergence |

---

## `SVDppModel` — SVD with Implicit Feedback

```python
class SVDppModel(SVDpp):
    def __init__(self, n_factors=100, n_epochs=30, lr_all=0.005, reg_all=0.05):
        SVDpp.__init__(...)
```

### The Extension

SVD++ adds an **implicit feedback** term to the user embedding:

$$\hat{r}_{u,i} = \mu + b_u + b_i + \mathbf{q}_i^\top \left(\mathbf{p}_u + |N(u)|^{-\frac{1}{2}} \sum_{j \in N(u)} \mathbf{y}_j\right)$$

Where:
- **N(u)**: the set of items user u has rated (regardless of the actual ratings)
- **yⱼ ∈ ℝᵏ**: an implicit vector for item j — learned parameter
- **|N(u)|^{-½}**: normalisation by the square root of the number of rated items

**Intuition:** The fact that a user has *watched* a movie (even without rating it) says something about their taste. A user who watched many sci-fi films implicitly signals sci-fi preference even if they rated some poorly. The implicit term captures this from interaction history alone.

### Trade-off vs. SVD

- **Better RMSE:** SVD++ (f=150) achieves 0.7890 vs. SVD 0.7913
- **3× slower training:** The implicit sum ∑yⱼ must be recomputed for each user at each gradient step
- **Higher overfitting risk:** The yⱼ vectors add extra degrees of freedom → need higher λ (0.05 vs 0.02 for SVD)
- **Amplifies popularity bias:** Popular items appear in many users' N(u), reinforcing their yⱼ vectors

---

## `NMFModel` — Non-Negative Matrix Factorisation

```python
class NMFModel(NMF):
    def __init__(self, n_factors=15, n_epochs=50, biased=True, reg_pu=0.06, reg_qi=0.06):
        NMF.__init__(...)
```

### The Key Constraint

NMF decomposes R ≈ W × H with **W, H ≥ 0** (all values non-negative).

**Why this matters:**
- In SVD, factors can cancel each other: a positive "action preference" can be offset by a negative "romance aversion"
- In NMF, factors are **purely additive**: each factor contributes positively. This creates "parts-based" representations — factor 1 might represent "comedy taste", factor 2 "sci-fi taste", etc.
- This makes the factors more interpretable, but limits expressiveness

### Stability Issue

NMF is sensitive to the number of factors:
- f=10 → CV RMSE = 0.8229 (stable)
- f=15 with λ=0.02 → CV RMSE = 0.95 ± 0.22 (unstable! — non-negativity interacts badly with unconstrained gradients)
- f=15 with λ≥0.08 → stable but RMSE > 0.90

This instability is why we cap NMF at f=10 in practice.

---

> [!WARNING]+ ❓ Assessor Question — What does the dot product pᵤ·qᵢ actually represent?
> The dot product is a measure of alignment between two vectors: high positive value = user u's taste aligns well with item i's characteristics. Concretely, if the k-th latent dimension represents "action preference", then pᵤ[k] encodes how much user u likes action, and qᵢ[k] encodes how much item i is associated with that dimension. The dot product sums all k dimension contributions — it's the total "compatibility score" between the user's preference profile and the item's characteristic profile.

> [!WARNING]+ ❓ Assessor Question — Why does SVD++ need a higher learning rate (η=0.01 instead of 0.005)?
> SVD++ has more parameters than SVD (the extra yⱼ vectors). With more parameters, each gradient update moves the model less than the same update in SVD — effectively, the "step size" needed to make progress is larger. At η=0.005 (SVD's optimal rate), SVD++ converges too slowly and 30 epochs aren't enough to reach its optimum. Raising to η=0.01 brings SVD++ to the same convergence speed as tuned SVD. The early evaluation at η=0.005 gave SVD++ RMSE=0.7930, masking its true advantage of 0.7890 at η=0.01.

> [!WARNING]+ ❓ Assessor Question — What is the difference between L1 and L2 regularisation?
> **L2 regularisation** (Ridge, used here): penalises the sum of squared weights. Forces weights toward zero smoothly — all weights become small but rarely exactly zero. Encourages *distributed* representations where many factors contribute a little. **L1 regularisation** (Lasso): penalises the sum of absolute weights. Forces some weights to exactly zero — creates sparse solutions. L1 would make many user/item embedding dimensions exactly zero. We use L2 because SGD with L2 is well-understood, converges reliably, and prevents extreme embeddings without enforcing sparsity (which would hurt expressiveness for latent factor models).

---

**See also:** [[Workflow/05_Hyperparameter_Tuning]] | [[Codes/07_Models_Hybrid]] | [[Codes/04_Models_Collaborative]]
