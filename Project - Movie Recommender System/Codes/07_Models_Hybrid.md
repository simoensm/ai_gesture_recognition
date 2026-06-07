---
tags:
  - recommender-systems
  - code
  - python
  - hybrid
  - MLSMM2156
---
# Hybrid Model

**File:** `recsys/models.py` — `HybridSVD` (line 1297)

The best-performing model. CV RMSE = **0.7308 ± 0.0015**.

---

## The Concept

A **linear blend** of two orthogonal signals:

$$\hat{r}_{u,i}^{\text{hybrid}} = \alpha \cdot \hat{r}_{u,i}^{\text{SVD}} + (1-\alpha) \cdot \hat{r}_{u,i}^{\text{content}}$$

| Signal | Captures | Best α |
|---|---|---|
| SVD (collaborative) | How users collectively rate movies | 0.2 (20%) |
| Content (genome Ridge) | What kind of movie it is, personalised per user | 0.8 (80%) |

The two signals are **orthogonal**: genome tags encode content type; SVD encodes residual collective rating patterns not captured by any tag. Adding 20% SVD corrects the errors the content model makes on movies that are underrated/overrated relative to their content profile.

---

## Code

```python
class HybridSVD(AlgoBase):
    def __init__(self, alpha=0.5, features_method="genome_full",
                 regressor_method="ridge", n_factors=100,
                 n_genome_components=150, lr_all=0.010, reg_all=0.05,
                 n_epochs=30, content_alpha=20):
        
        self.alpha = alpha     # SVD weight
        
        # Two independent sub-models
        self.svd = SVD(n_factors=n_factors, lr_all=lr_all,
                       reg_all=reg_all, n_epochs=n_epochs, random_state=42)
        self.content = ContentBasedWithStats(
            features_method, regressor_method, n_genome_components,
            alpha=content_alpha
        )
```

Two sub-models are created at `__init__` time:
- `self.svd`: a Surprise `SVD` instance (the latent factor component)
- `self.content`: a `ContentBasedWithStats` instance (the genome Ridge component)

Note: `alpha` is the hybrid blending parameter. `content_alpha` is the Ridge regularisation for the content arm. Two different alphas, one in the hybrid context, one inside Ridge — don't confuse them.

---

### `fit(trainset)`

```python
def fit(self, trainset):
    AlgoBase.fit(self, trainset)
    self.svd.fit(trainset)       # train SVD on the same fold
    self.content.fit(trainset)   # train Ridge for each user on the same fold
    return self
```

Both sub-models are trained on the **same training fold**. This is important: they see the same data, so neither has an advantage from more information. The blend of their predictions is evaluated on the same held-out test fold.

---

### `estimate(u, i)`

```python
def estimate(self, u, i):
    try:
        svd_score = self.svd.estimate(u, i)
    except PredictionImpossible:
        svd_score = self.trainset.global_mean    # fallback: global mean
    
    try:
        content_score = self.content.estimate(u, i)
    except PredictionImpossible:
        content_score = self.trainset.global_mean  # fallback: global mean
    
    return self.alpha * svd_score + (1 - self.alpha) * content_score
```

The `PredictionImpossible` exception is caught for both arms:
- SVD raises it if user or item is unknown
- Content raises it if the item has no features or the user profile wasn't fitted

**Cold start handling:** For a completely new item (no ratings, thus no SVD factors), SVD falls back to the global mean. But if the item has genome tags, the content arm still personalises. This makes the hybrid more robust to the item cold-start problem than pure SVD.

---

## Why α=0.2 is Optimal

From the alpha sweep (full results in [[Workflow/05_Hyperparameter_Tuning]]):

```
α=0.0  →  pure content:  RMSE = 0.7346
α=0.2  →  hybrid:        RMSE = 0.7308  ← best
α=0.5  →  equal blend:   RMSE = 0.7398
α=1.0  →  pure SVD:      RMSE = 0.7913
```

The content model already captures ~80% of the predictable variance. SVD adds a small but consistent correction. At α=0.2, the SVD weight is large enough to provide its orthogonal signal but small enough not to erode the content model's precision.

**Why does this work mathematically?** If ε_content and ε_SVD are the errors of the two models, and if their errors are approximately uncorrelated (orthogonal signals), then:

$$\text{Var}(\alpha \cdot \epsilon_\text{SVD} + (1-\alpha) \cdot \epsilon_\text{content}) < (1-\alpha)^2 \cdot \text{Var}(\epsilon_\text{content})$$

as long as α > 0 and Cov(ε_SVD, ε_content) < Var(ε_SVD). This is the mathematical reason for the improvement.

---

## The Full Hybrid Config (Default)

```python
# From configs.py (WebappConfig) — production settings
HybridSVD(
    alpha=0.2,               # 20% SVD, 80% content
    features_method="genome_full",
    regressor_method="ridge",
    content_alpha=20,        # Ridge L2 strength
    n_factors=100,           # SVD latent dimensions
    reg_all=0.05,            # SVD L2 regularisation (tuned)
    lr_all=0.010,            # SVD learning rate (tuned)
    n_epochs=30              # SGD passes
)
```

---

> [!WARNING]+ ❓ Assessor Question — Why does the hybrid beat both standalone models?
> The two components make errors on **different** movies. The content model errs when a movie's rating differs from what its tags would predict (e.g., a technically good drama that users nonetheless rated poorly due to poor execution). The SVD model errs on movies with unusual content profiles (e.g., avant-garde films that don't fit the collaborative patterns). When the errors are uncorrelated, blending the two predictors reduces total variance. The optimal weight (0.2 for SVD) reflects the relative error magnitude: SVD has 3× higher standalone RMSE, so we trust it less.

> [!WARNING]+ ❓ Assessor Question — What is Burke's (2002) cascade hybrid architecture?
> Burke (2002) defined several hybrid architectures. A **cascade hybrid** uses one recommender to produce a candidate list, then a second to re-rank it. A **weighted hybrid** (what we use here) linearly blends predictions from multiple models. The cascade architecture is used in the **webapp's cold-start logic**: Phase 0 (cold start) → Phase 1 (weighted blend with popularity) → Phase 2 (full configured model). See [[Webapp/02_Server_Endpoints]].

> [!WARNING]+ ❓ Assessor Question — Does the hybrid handle item cold start better than SVD?
> Yes. For a new item with no ratings (pure cold start), SVD produces no useful embedding — it falls back to the global mean. But if the item has genome tags (which are available independently of ratings), the content arm can still personalise. In the hybrid at α=0.2, the prediction becomes 0.2 × global_mean + 0.8 × personalised_content_score — much better than pure SVD's global mean, and still somewhat personalised.

---

**See also:** [[Codes/05_Models_Latent_Factor]] | [[Codes/06_Models_Content_Based]] | [[Workflow/08_Results_and_Decision_Making]]
