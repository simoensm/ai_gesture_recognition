---
tags: [recommender-systems, bias, statistics, collaborative-filtering, bayesian]
---
# Rating Bias and Statistical Corrections

**Related:** [[Codes/06_Models_Content_Based]] | [[Workflow/04_Feature_Engineering]] | [[Concepts/Collaborative_Filtering]]

---

## 1. Mild Positive Bias

Users tend to rate things they _chose_ to watch, and they often rate things they liked. This creates **selection bias**: the average rating is skewed above the neutral midpoint (e.g., mean rating on MovieLens ≈ 3.5 out of 5, not 3.0).

Some users are also systematically generous (always rate 4–5) or harsh (always rate 1–3). This is **per-user bias**.

---

## 2. Per-User Mean-Centering (Deviation Scores)

In collaborative filtering, instead of working with raw ratings $r_{ui}$, subtract each user's mean:

$$\tilde{r}_{ui} = r_{ui} - \bar{r}_u$$

This converts ratings to **deviation scores** — how much the user liked this item _relative to their own average_. A score of +1.5 means "they liked it 1.5 stars more than usual."

**Why?** Without this, a harsh user who gives 3 to everything looks similar to a generous user who gives 3 to something they hated. Mean-centering removes the systematic per-user offset so you're comparing true preferences.

This is why Pearson/adjusted-cosine outperforms MSD in the ablation: **CV RMSE 0.8280 vs 0.8371**.

---

## 3. Bayesian Shrinkage in Quality Statistics

Suppose a movie has only 2 ratings, both 5 stars. Should you trust it's a 5/5 movie? No — small samples are noisy.

**Bayesian shrinkage** pulls uncertain estimates toward a prior (the global mean $\mu$):

$$\hat{r}_i = \frac{n_i \cdot \bar{r}_i + \kappa \cdot \mu}{n_i + \kappa}$$

where:
- $n_i$ = number of ratings for item $i$
- $\bar{r}_i$ = observed mean for item $i$
- $\kappa$ = shrinkage strength (default = 25 in the project)

With few ratings ($n_i \ll \kappa$), the estimate collapses toward $\mu$. With many ratings, the observed mean dominates.

**Why?** The positive bias makes raw averages unreliable for items with few ratings. A movie with 2 five-star ratings looks better than one with 1000 four-star ratings without shrinkage.

**Implementation in `ContentBasedWithStats`:**

```python
item_bayesian_mean = (n * item_mean + 25 * global_mean) / (n + 25)
```

---

## 4. Similarity Metrics Compared

### MSD — Mean Squared Difference

$$\text{sim}(u,v) = \frac{1}{\frac{1}{n}\sum(r_{ui} - r_{vi})^2 + 1}$$

- Measures **rating disagreement** on co-rated items
- Sensitive to scale: a user who gives 1s and 5s looks "far" from one who gives 2s and 4s even if they agree on ranking

### Pearson / Cosine on Mean-Centered Vectors

$$\text{sim}(u,v) = \frac{(\mathbf{r}_u - \mu_u) \cdot (\mathbf{r}_v - \mu_v)}{\|\mathbf{r}_u - \mu_u\| \cdot \|\mathbf{r}_v - \mu_v\|}$$

- Subtracts each user's own mean → removes rating-scale bias
- Best in the user-based CF ablation: **RMSE 0.8280**

### Adjusted Cosine (item-based)

$$\text{sim}(i,j) = \frac{\sum_u (r_{ui} - \mu_u)(r_{uj} - \mu_u)}{\|\cdots\| \cdot \|\cdots\|}$$

- Subtracts the **user** mean (not item mean) — critical for items
- A blockbuster rated 4 by everyone looks "similar" to a niche film rated 4 by fans — centering on the user reveals whether each was above/below _that user's_ baseline

### Jaccard

$$\text{sim}(u,v) = \frac{|\text{items rated by both}|}{|\text{items rated by either}|}$$

- Ignores rating values — pure engagement/overlap signal
- Weaker when preference information is available

### Spearman

$$\text{sim}(u,v) = 1 - \frac{6\sum d^2}{n(n^2-1)}$$

- Treats ratings as **ordinal** (order matters, not exact gap)
- Robust to outliers; matched Pearson in the ablation (RMSE 0.8335 vs 0.8280)

---

## 5. Why the Best Model Does Not Use Similarity

The best model (`ContentBasedWithStats`, CV RMSE 0.7344) uses **Ridge regression**, not similarity. Instead of asking "who is similar to this user?", it asks: "given this user's rating history, what linear combination of genome features predicts their ratings?"

This sidesteps the core weaknesses of similarity-based CF:
- **Sparsity** — two users rarely rated the same movies → noisy similarity estimates
- **Cold start** — new users have no co-rated items → similarity undefined
- **Scalability** — computing all user-pair similarities is $\mathcal{O}(n^2)$

Similarity metrics are conceptually elegant and easy to explain, but Ridge on content features generalises better when the rating matrix is sparse (MovieLens density ≈ 4%).
