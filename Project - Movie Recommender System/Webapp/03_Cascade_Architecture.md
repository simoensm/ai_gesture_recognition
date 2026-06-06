---
tags: [recommender-systems, webapp, cascade, cold-start, hybrid]
---
# Cascade Hybrid Architecture

The server implements a **3-phase cascade** that transitions from cold-start to full personalisation based on how many explicit ratings the user has given. The active Phase 2 model is controlled by `WebappConfig` in `configs.py`.

**See also:** [[Webapp/01_Webapp_Overview]] | [[Webapp/02_Server_Endpoints]] | [[Codes/07_Models_Hybrid]] | [[Papers/Burke2002_HybridRS]]

---

## Phase 0 — Cold Start (0 explicit ratings)

Triggered when the user has rated nothing yet (right after onboarding).

**Method:** Content-based Ridge on preference pseudo-ratings.

Onboarding preferences (genres, directors, actors, genome tags, era) are converted to **synthetic item ratings** via `_prefs_to_pseudo_ratings()`:

| Signal | Weight |
|---|---|
| Genre match | +0.8 per genre (cap +2.0) |
| Director hit | +1.5 |
| Actor match | +0.8 per actor (cap +1.6) |
| Genome tag anchor (relevance > 0.80) | +0.6 |
| Preferred era | +0.3 (soft boost, no exclusion) |

Scores are normalised to [3.5, 5.0]. Then `Ridge(α=20)` is fitted on those pseudo-rated items using the `genome_full` feature matrix (1,128 dimensions). Predicted scores for all items are ranked, with an optional +0.2 era boost.

**Fallback** (< 3 anchor items): genre-filtered popularity ranking.

---

## Phase 1 — Grey Start (1–19 explicit ratings)

**Method:** Blended content + popularity.

$$\text{score} = \alpha \cdot \text{content\_score} + (1-\alpha) \cdot \text{popularity\_score}$$
$$\alpha = \frac{n_\text{ratings}}{20} \quad \text{(grows from 0.05 to 0.95)}$$

The content model is fitted on **actual ratings + pseudo-ratings merged** (actual takes precedence), so onboarding preferences keep steering until the user has enough signal. Popularity is pre-computed as inverse-rank normalised to [0.5, 5.0].

---

## Phase 2 — Warm Start (≥ 20 explicit ratings)

The **configured model** from `configs.py`. Currently active:

**`ContentBasedWithStats` + `genome_full` + Ridge(α=20)**

At every `/recommend` call:

1. The 1,128 genome tag relevance scores are loaded (once at startup).
2. Four item quality statistics are appended: Bayesian mean, log-count, std, high-ratio — each MinMaxScaled to [0,1].
3. A fresh `Ridge(α=20)` is **fitted on-the-fly** on the user's rated items only.
4. The fitted regressor predicts scores for all catalogue items; top-N are returned.

**Other Phase 2 options** (available in `configs.py`):

| `MODEL_CLASS` | Method | Prediction formula |
|---|---|---|
| `UserBased` | User-user CF, Pearson, k=40 | $\hat{r}(u,i) = \mu_u + \frac{\sum \text{sim} \cdot (r_{vi} - \mu_v)}{\sum \text{sim}}$ |
| `ItemBased` | Item-item CF, adjusted cosine via TruncatedSVD (100 factors), k=20 | $\hat{r}(u,i) = \mu_i + \frac{\sum \text{sim} \cdot (r_{uj} - \mu_j)}{\sum \text{sim}}$ |
| `SVD` | Surprise SVD (100 factors, 30 epochs); new user's $p_u$ estimated via Ridge at request time | $\hat{r} = \mu + \hat{b}_u + b_i + \hat{p}_u \cdot q_i$ |

---

## `/similar` Endpoint

Item-to-item cosine similarity computed on the active feature space (`_feat_matrix`) — genome vectors for content models, item latent factors for SVD/ItemBased. Powers the "Similar movies" UI feature.

---

## ContentBasedWithStats — How It Works

**Base `ContentBased`** fits a Ridge regressor per user:

$$\mathbf{w}_u^* = \arg\min_{\mathbf{w}} \sum_{i \in \mathcal{I}_u} (r_{u,i} - \mathbf{w}^\top \mathbf{x}_i)^2 + \alpha|\mathbf{w}|^2$$

where $\mathbf{x}_i$ is the 1,128-dimensional genome tag vector. **Fundamental limit:** genome tags encode _what kind_ of movie it is, not _how good_ it is. Two atmospheric thrillers with nearly identical genome vectors can have very different quality.

**ContentBasedWithStats** appends 4 quality statistics to $\mathbf{x}_i$, computed exclusively from training ratings (no leakage):

| Stat | Formula | What it captures |
|---|---|---|
| `item_bayesian_mean` | $(n \cdot \mu_i + 25 \cdot \mu_\text{global}) / (n + 25)$ | Shrinkage-corrected quality |
| `item_log_count` | $\log(1 + n)$ | Popularity proxy |
| `item_std` | $\sigma(r_i)$ | Divisiveness / polarisation |
| `item_high_ratio` | $P(r \geq 4)$ | "Loved it" rate |

All four are MinMaxScaled to [0,1] to match the genome feature range.

**The improvement:** 0.7353 → **0.7344 RMSE** (0.001 gain, zero extra inference cost).

**Why Ridge over other models:**

| Alternative | Problem |
|---|---|
| OLS | Overfits, near-singular matrix under 1,128 correlated features |
| RidgeCV | No benefit, slower than fixed α=20 |
| Lasso | Zeros out correlated genome tags → spurious sparsity |
| Random/popularity | Ignores user preferences entirely |

> *"Ridge is preferred over OLS (unstable under correlated high-dimensional features), Lasso (spurious sparsity on jointly informative genome tags), and RidgeCV (empirically equivalent but slower). The closed-form solution enables real-time per-request inference in the webapp."*
