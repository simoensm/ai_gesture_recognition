- streamlit : plus simple (vitesse)
- flask/jango : de base need une formation mais avec l'IA, facilement 
- Genome DNA Match visualisation
- 3 top redondants
- * pour mettre subsection invisible sur la tdm
- hybrid SVD to ablation and stat test X
- Figures to add X
	- 1. Bar chart: best RMSE per family (genome dominance immediately visible)
	1. Ridge α sweep: U-shaped regularization curve
	2. SVD++ sweep: RMSE vs n_factors line plot
	3. ILD vs RMSE scatter: diversity-accuracy trade-off plot with one point per model family
1. ILD and novelty discussions (hit rate) X
2. Genomes > TMDB but what happens for movies with **no genome coverage** X
3. LEakage probably with items stats scaled (trouver solution car sinn c har)
4. README to update
5. he abstract's comparison now cites `KNNWithMeans Pearson k=80: 0.8280` as the best CF model, but this is a built-in Surprise model with no novelty — your custom Spearman was the creative contribution. The abstract framing slightly undersells your own work by choosing the metric winner over the methodological contribution.
6. ITR and AMI = stats ???
7. Random splitting versus temp splitting ? 
8. The paper states RMSE is preferred over MAE (supports our choice). It also says coverage/novelty "cannot be obtained offline" — we do compute them offline.
## Cascade Hybrid Architecture (Burke 2002)

The server implements a **3-phase cascade** that transitions from cold-start to full personalisation based on how many ratings the user has given. The active Phase 2 model is controlled by `WebappConfig` in [configs.py](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/configs.py).

---

### Phase 0 — Cold Start (0 explicit ratings)

Triggered when the user has rated nothing yet (right after onboarding).

**Method:** Content-based Ridge on preference pseudo-ratings.

The onboarding preferences (genres, directors, actors, genome tags, era) are converted to **synthetic item ratings** via `_prefs_to_pseudo_ratings()` using additive scoring:

|Signal|Weight|
|---|---|
|Genre match|+0.8 per genre (cap +2.0)|
|Director hit|+1.5|
|Actor match|+0.8 per actor (cap +1.6)|
|Genome tag anchor (relevance > 0.80)|+0.6|
|Preferred era|+0.3 (soft boost, no exclusion)|

Scores are normalised to [3.5, 5.0]. Then a `Ridge(α=20)` is fitted on those pseudo-rated items using the `genome_full` feature matrix (1,128 dimensions). The predicted scores for all items are ranked, with an optional +0.2 era boost.

**Fallback** (< 3 anchor items): genre-filtered popularity ranking.

---

### Phase 1 — Grey Start (1–19 explicit ratings)

**Method:** Blended content + popularity.

```
score = α · content_score + (1−α) · popularity_score
α = n_ratings / 20   →   grows from 0.05 to 0.95
```

The content model is fitted on **actual ratings + pseudo-ratings merged** (actual takes precedence), so onboarding preferences keep steering until the user has enough signal. Popularity is pre-computed as inverse-rank normalised to [0.5, 5.0].

---

### Phase 2 — Warm Start (≥ 20 explicit ratings)

The **configured model** from [configs.py:91-98](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/configs.py#L91-L98). Currently active:

**`ContentBasedWithStats` + `genome_full` + Ridge(α=20)**

At every `/recommend` call:

1. The 1,128 genome tag relevance scores are loaded (once at startup).
2. Four item quality statistics are appended: Bayesian mean, log-count, std, high-ratio — each MinMaxScaled to [0,1].
3. A fresh `Ridge(α=20)` is **fitted on-the-fly** on the user's rated items only.
4. The fitted regressor predicts scores for all catalogue items; top-N are returned.

The other Phase 2 model options available (commented out in configs.py) are:

|`MODEL_CLASS`|Method|Prediction formula|
|---|---|---|
|`UserBased`|User-user CF, cosine over mean-centered ratings (Pearson), k=40|r̂(u,i) = μ_u + Σ sim·(r_vi − μ_v) / Σ sim|
|`ItemBased`|Item-item CF, adjusted cosine approximated via TruncatedSVD (100 factors), k=20|r̂(u,i) = μ_i + Σ sim·(r_uj − μ_j) / Σ sim|
|`SVD`|Surprise SVD trained at startup (100 factors, 30 epochs); new user's latent vector p_u estimated via Ridge at request time|r̂ = μ + b̂_u + b_i + p̂_u · q_i|

---

### `/similar` Endpoint (always available)

Item-to-item cosine similarity computed on the active feature space (`_feat_matrix`) — genome vectors for content models, item latent factors for SVD/ItemBased. Used by the "Similar movies" UI feature.


## How ContentBasedWithStats works

Start with the base model to understand what it adds.

### ContentBased (base)

For each user independently, fits a Ridge regressor on their rated items:

$$\mathbf{w}_u^* = \arg\min_{\mathbf{w}} \sum_{i \in \mathcal{I}_u} (r_{u,i} - \mathbf{w}^\top \mathbf{x}_i)^2 + \alpha |\mathbf{w}|^2$$

where **x**_i is the 1,128-dimensional genome tag vector. The weight vector **w**_u learns which perceptual dimensions matter to _this specific user_. If you rate atmospheric thrillers highly, those genome dimensions get large weights. Prediction for a new item is just a dot product: **w**_u · **x**_i.

**The fundamental limit:** genome tags encode _what kind_ of movie it is. They cannot encode _how good_ it is. Two atmospheric thrillers can have nearly identical genome vectors — one is universally loved, one is mediocre — and the model has no way to distinguish them.

---

### What ContentBasedWithStats adds

It appends **4 item quality statistics** to **x**_i at each training fold, computed _exclusively from the training ratings_ (no leakage):

|Stat|Formula|What it captures|
|---|---|---|
|`item_bayesian_mean`|$(n \cdot \mu_i + 25 \cdot \mu_{\text{global}}) / (n + 25)$|Shrinkage-corrected item quality — for films with few ratings, shrinks toward global mean instead of trusting a noisy estimate|
|`item_log_count`|$\log(1 + n)$|Popularity proxy — log scale prevents blockbusters from dominating|
|`item_std`|$\sigma(r_i)$|Divisiveness — high std means polarizing film (Interstellar-type love/hate split)|
|`item_high_ratio`|$\mathbb{P}(r \geq 4)$|"Loved it" rate — orthogonal to the mean (a 3.5 mean film can still have 40% four-star ratings or 10%)|

All four are MinMaxScaled to [0,1] to match the genome feature range, so Ridge treats them at the same scale.

**The key detail:** these are computed from `trainset.ir` inside `fit()`, reset at every fold. The global `_item_stats_scaled()` function that uses `load_ratings()` (full dataset) is a _different function_ and would be leaky — `ContentBasedWithStats` deliberately bypasses it.

---

### Is it pertinent here?

**Yes, and for a concrete reason.** The genome + quality stats combination covers two orthogonal information axes:

- **Genome** → _what kind_ (atmosphere, pace, theme, tone)
- **Stats** → _how good globally_ (community consensus quality)

Ridge learns to use both. For a user who likes sci-fi, the genome weights will emphasize sci-fi tags. But among all sci-fi films, the Bayesian mean will push Blade Runner (loved by most) above a generic sci-fi B-movie that happens to hit the same tags.

**The numbers confirm it:** 0.7353 (genome only) → 0.7344 (+ stats), a consistent 0.001 RMSE gain across all 5 folds with **zero extra inference cost** — the stats are just 4 extra columns in the feature matrix, already there when Ridge predicts.

**For the LOO hit rate** you're about to run, ContentBasedWithStats is likely to outperform ContentBased on ranking quality too. LOO asks "did the model rank the withheld item in the top 40?" — the Bayesian mean stat directly helps here because genuinely good movies will score higher for every user.

**The only honest caveat:** the gain is small (0.001 RMSE ≈ 0.1%). This is because genome features already partially encode quality indirectly — highly-rated films tend to have denser, more reliable genome annotations. The stats aren't fully orthogonal in practice, which limits the gain. But free 0.001 RMSE improvements are still worth taking, especially in a hackathon.

It appends 4 simple facts about each movie — facts that have nothing to do with content, only with how other people rated it:

**1. "Is this movie generally well-rated?"** The average score, adjusted so movies with only 3 ratings don't get unfairly boosted. A movie with 800 ratings averaging 4.1 is more trustworthy than a movie with 3 ratings averaging 5.0.

**2. "How many people rated it?"** Popular movies. More ratings = more evidence it's worth watching.

**3. "Do people agree about it?"** Some movies divide people — half love it, half hate it (like Marmite). Others everyone agrees on. This tells the model how "safe" a recommendation is.

**4. "What fraction of people really loved it?"** Not just the average, but specifically: how many people gave it 4 stars or more? A movie with average 3.5 where 60% of people gave 4+ is different from one where only 20% did.
The improvement is real but small: **0.7353 → 0.7344 RMSE**. About 0.001 better.

That's because genome tags already _partially_ capture quality — beloved films tend to have richer, more reliable genome annotations. The stats aren't telling the model something completely new, just filling a small gap.

The gain is tiny but it costs nothing extra at prediction time — the 4 numbers are just extra columns the model already has in its spreadsheet.

## What are we comparing?

In collaborative filtering, you have a **ratings matrix** R where rows = users, columns = movies, cells = ratings (or empty).

```
         Inception  Matrix  Titanic  Shrek
Alice         5       4       ?       2
Bob           4       5       ?       1
Carol         2       1       5       4
Dave          ?       4       2       3
```

You want to predict Alice's rating for Titanic. You have two options:

**User-based** — find users _similar to Alice_, then borrow their Titanic rating:

> "Bob likes the same films as Alice → Bob rated Titanic 3 → predict ~3"

**Item-based** — find movies _similar to Titanic_, then use Alice's ratings on those:

> "Titanic is similar to Shrek (both scored low by Alice) → predict low"

**You are comparing rating vectors, not content.** The similarity metric measures how two users (or two items) vote alike across the movies they both rated.

---

## Why do we need a metric at all?

Because "similar" is vague. You need a number. Each metric makes a different assumption:

### MSD — Mean Squared Difference

```
sim(u,v) = 1 / (Σ(r_ui − r_vi)² / n  +  1)
```

- Measures **rating disagreement** on co-rated items
- The `+1` keeps it in (0, 1] and avoids division by zero
- Sensitive to scale: user who gives 1s and 5s looks "far" from one who gives 2s and 4s even if they agree on ranking
- Fast, symmetric, interpretable

### Pearson / Cosine on mean-centered vectors

```
sim(u,v) = (r_u − μ_u) · (r_v − μ_v) / (‖r_u − μ_u‖ · ‖r_v − μ_v‖)
```

- Subtracts each user's **own mean** first → removes rating-scale bias
- Alice gives 3/5 on average, Bob gives 4/5 → after centering, both a "3" means the same ("below my average")
- This is why it beats MSD in your ablation: **CV RMSE 0.8280 vs 0.8371**

### Adjusted cosine (item-based only)

```
sim(i,j) = Σ_u (r_ui − μ_u)(r_uj − μ_u) / (‖…‖ · ‖…‖)
```

- Same idea but subtracts the **user** mean, not the item mean
- Sarwar (2001) showed this is critical for items: a blockbuster rated 4 by everyone looks "similar" to a niche film rated 4 by fans, but they're actually very different — centering on the user who rated both reveals whether they were above or below _that user's_ baseline

### Jaccard

```
sim(u,v) = |items rated by both| / |items rated by either|
```

- Ignores rating values entirely — only cares about **overlap of rated sets**
- Useful when ratings are noisy or sparse; pure engagement signal
- Weaker than Pearson in your results because it throws away preference information

### Spearman

```
sim(u,v) = 1 − 6Σd² / n(n²−1)    where d = rank difference
```

- Treats ratings as **ordinal** (order matters, not the exact gap between 1 and 2)
- Robust to outliers; a 5-star vs 1-star rant matters less than the ranks
- Matched Pearson in your ablation (CV RMSE 0.8335 vs 0.8280), slightly worse

---

## Why does this matter for prediction?

Once you have similarities, the prediction formula is:

```
r̂(u,i) = μ_u  +  Σ_{v ∈ top-k} sim(u,v) · (r_vi − μ_v)
                   ────────────────────────────────────────
                         Σ_{v ∈ top-k} sim(u,v)
```

The metric determines **who gets into top-k** and **how much weight they carry**. A bad metric → wrong neighbours → wrong weights → wrong predictions → high RMSE.

---

## Why do we NOT use similarity in the best model?

Your best model (`HybridSVD`, CV RMSE 0.7308) uses **Ridge regression**, not similarity. Instead of asking "who is similar to this user?", it asks:

> "Given this user's rating history, what linear combination of genome features predicts their ratings?"

This sidesteps the core weaknesses of similarity-based CF:

- **Sparsity** — two users rarely rated the same movies, so similarity estimates are noisy
- **Cold start** — new users have no co-rated items; similarity is undefined
- **Scalability** — computing all user-pair similarities is O(n²)

Similarity metrics are conceptually elegant and easy to explain, but Ridge on content features generalises better when the rating matrix is sparse — which MovieLens 1M is (density ≈ 4%).

## 1. "Mild Positive Bias"

Users tend to rate things they _chose_ to watch, and they often rate things they liked. This creates **selection bias**: the average rating is skewed above the neutral midpoint (e.g., the mean rating on MovieLens is ~3.5 out of 5, not 3.0).

Some users are also systematically generous (always rate 4-5) or harsh (always rate 1-3). This is **per-user bias**.

---

## 2. Per-User Mean-Centering (Deviation Scores)

In collaborative filtering (e.g., user-based KNN, matrix factorization), instead of working with raw ratings $r_{ui}$, you subtract each user's mean:

$$\tilde{r}_{ui} = r_{ui} - \bar{r}_u$$

This converts ratings to **deviation scores** — how much the user liked this item _relative to their own average_. A score of +1.5 means "they liked it 1.5 stars more than usual."

**Why?** Without this, a harsh user who gives 3 to everything looks similar to a generous user who gives 3 to something they hated. Mean-centering removes that systematic per-user offset so you're comparing true preferences.

---

## 3. Bayesian Shrinkage in Quality Statistics

Suppose a movie has only 2 ratings, both 5-stars. Should you trust it's a 5/5 movie? No — small samples are noisy.

**Bayesian shrinkage** pulls uncertain estimates toward a prior (typically the global mean $\mu$):

$$\hat{r}_i = \frac{n_i \cdot \bar{r}_i + \kappa \cdot \mu}{n_i + \kappa}$$

Where:

- $n_i$ = number of ratings for item $i$
- $\bar{r}_i$ = observed mean for item $i$
- $\kappa$ = shrinkage strength (how much you trust the prior vs. the data)

With few ratings ($n_i \ll \kappa$), the estimate collapses toward $\mu$. With many ratings, the observed mean dominates.

**Why?** Because the positive bias makes raw averages unreliable for items with few ratings — a movie with 2 five-star ratings looks better than one with 1000 four-star ratings without shrinkage.

**1. LOO forces predictions over the entire anti-testset**

In `02_evaluator.ipynb`, `generate_loo_top_n()` uses `LeaveOneOut`, which for every user withholds one rating and then scores **all unrated items** to build the top-N list. For ~1,000 users × ~8,744 movies that is ≈ **8.7 million predictions**, just to compute hit rate.

The ablation study never does LOO. For diversity metrics it uses `_partial_anti_testset()` ([ablation_study.py:214](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/ablation_study.py#L214)), which samples:

- `N_DIV_USERS = 300` users (not all 1,000)
- `CAND_ITEMS_CB = 600` items per user for content-based models

So the ablation's equivalent step is at most **300 × 600 = 180,000 predictions** for CB — a **~50× reduction**.

---

**2. Content-based models are slow per prediction**

Each `ContentBased.estimate()` call does a `pandas.loc` lookup and a dot product per item. At 8.7M calls in the LOO path this is brutal; at 180K in the ablation path it's manageable. The comment in [configs.py:146](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/configs.py#L146) calls this out explicitly:

> _"content-based models are slow per prediction (pandas.loc per call)"_

---

**3. The evaluator runs three separate passes; the ablation runs one**

The evaluator computes:

- Split RMSE/MAE (one fit + 25% test)
- LOO hit rate (one fit + 8.7M anti-testset predictions)
- Full novelty (another pass over anti-testset)

The ablation runs **one 5-fold CV loop** that produces RMSE, MAE, ILD, coverage, and novelty all in a single anti-testset pass per fold — and that pass is sampled.

---

**Summary table:**

||Evaluator|Ablation Study|
|---|---|---|
|Accuracy|single 75/25 split|5-fold CV|
|Ranking eval|full LOO (~8.7M preds)|sampled anti-testset (~180K for CB)|
|Users scored|all ~1,000|300 (100 for CB)|
|Items per user|all unrated (~8,744)|600 cap for CB, full for CF|
|Passes|3|1 per fold|

The LOO hit rate is the expensive line item. If you want the ablation to be as rigorous as the evaluator on ranking quality, you'd have to accept the full anti-testset cost — or accept that hit rate is only sampled

1 ** LOO forces predictions over the entire anti-testset**

In `02_evaluator.ipynb`, `generate_loo_top_n()` uses `LeaveOneOut`, which for every user withholds one rating and then scores **all unrated items** to build the top-N list. For ~1,000 users × ~8,744 movies that is ≈ **8.7 million predictions**, just to compute hit rate.

The ablation study never does LOO. For diversity metrics it uses `_partial_anti_testset()` ([ablation_study.py:214](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/ablation_study.py#L214)), which samples:

- `N_DIV_USERS = 300` users (not all 1,000)
- `CAND_ITEMS_CB = 600` items per user for content-based models

So the ablation's equivalent step is at most **300 × 600 = 180,000 predictions** for CB — a **~50× reduction**.

---

**2. Content-based models are slow per prediction**

Each `ContentBased.estimate()` call does a `pandas.loc` lookup and a dot product per item. At 8.7M calls in the LOO path this is brutal; at 180K in the ablation path it's manageable. The comment in [configs.py:146](vscode-webview://1mlk172nvjebg3dhqdpn78h1o6n4lkiju4l6mfsoka6mpqsqmgl4/recsys/configs.py#L146) calls this out explicitly:

> _"content-based models are slow per prediction (pandas.loc per call)"_

---

**3. The evaluator runs three separate passes; the ablation runs one**

The evaluator computes:

- Split RMSE/MAE (one fit + 25% test)
- LOO hit rate (one fit + 8.7M anti-testset predictions)
- Full novelty (another pass over anti-testset)

The ablation runs **one 5-fold CV loop** that produces RMSE, MAE, ILD, coverage, and novelty all in a single anti-testset pass per fold — and that pass is sampled.

---

**Summary table:**

||Evaluator|Ablation Study|
|---|---|---|
|Accuracy|single 75/25 split|5-fold CV|
|Ranking eval|full LOO (~8.7M preds)|sampled anti-testset (~180K for CB)|
|Users scored|all ~1,000|300 (100 for CB)|
|Items per user|all unrated (~8,744)|600 cap for CB, full for CF|
|Passes|3|1 per fold|

The LOO hit rate is the expensive line item. If you want the ablation to be as rigorous as the evaluator on ranking quality, you'd have to accept the full anti-testset cost — or accept that hit rate is only sampled.

**`UserBased` is a slower Python reimplementation of `KNNWithMeans` whose only purpose is to unlock Spearman and Jaccard**. For cosine/pearson/msd you should always prefer `KNNWithMeans`, which is why in your `EvalConfig` those configurations use `KNNWithMeans` and only the Spearman/Jaccard configs use `UserBased`

1. `MovieContext.jsx` POSTs to `http://localhost:5001/recommend` with the user's ratings, watched history, and onboarding preferences (line 250).
    
2. `server.py` runs the chosen model — `fit()` + `predict()` on the fly for content-based and hybrid, or fold-in via Ridge for SVD — and returns `{recommendations: [...], matchScores: {...}}`.
    
3. The frontend stores the response in `serverResult` and uses `serverResult.recommendations` for ordering and `serverResult.matchScores` for the displayed match percentages (lines 431–449).
    
4. The **only client-side modification** on top of server scores is the mood adjustment (±10 pts max, Eq. mood in the report) and the genre boost cap (+15). The underlying ranking is always the model's.
    
5. There is a **fallback** (the pure client-side genre-boost heuristic) but it only activates when `serverOnline === false` — i.e. when the Flask server is not running. When the server is up, this fallback is never touched.
    

So the answer is unambiguous: when the server is running, every recommendation list you see is computed by whichever of the 4 models you selected at startup. The client only applies the bounded mood re-ranking on top

Relecture :
Abstract : OK
Introduction : OK
Data : OK
Theoretical : Under review
Exp Workflow : OK
Ablation Study & Results : 


## 1. Le Test de Friedman

### Principe

C'est l'équivalent non-paramétrique d'une **ANOVA à mesures répétées**. Il ne compare pas les RMSE directement — il compare les **rangs** à chaque fold.

**Étape 1 — Ranger les modèles à chaque fold :**

```
          Fold 1   Fold 2   Fold 3   Rang moyen
Hybrid     0.728    0.731    0.729   → rang 1.0
SVD++      0.787    0.791    0.789   → rang 2.0
ItemBased  0.823    0.827    0.825   → rang 3.0
```

**Étape 2 — Calculer la statistique :**

$$\chi^2_F = \frac{12N}{k(k+1)} \sum_{j=1}^{k} \left(\bar{r}_j - \frac{k+1}{2}\right)^2$$

- $N$ = nombre de folds
- $k$ = nombre de modèles
- $\bar{r}_j$ = rang moyen du modèle $j$

**Étape 3 — Comparer à une loi $\chi^2$ à $k-1$ degrés de liberté.**

Si $p < 0.05$ → on rejette $H_0$ : **"tous les modèles sont équivalents"**

### Vos résultats

|Test|$\chi^2$|$p$|Conclusion|
|---|---|---|---|
|Cross-family (3 modèles, 3 folds)|6.00|**0.050**|Significatif — les familles ne sont pas équivalentes|
|Within content (5 modèles, 5 folds)|17.92|**0.00128**|Très significatif|
|Within latent factor (5 modèles, 5 folds)|20.00|**0.00050**|Très significatif|
|Within hybrid alpha (9 configs, 5 folds)|39.79|**0.000004**|Hautement significatif|

---

## 2. Le Test Post-Hoc de Nemenyi

### Pourquoi en a-t-on besoin ?

Friedman dit "au moins un modèle est différent" — mais **lequel ?** Nemenyi compare toutes les paires.

### Principe

On calcule une **Critical Difference (CD)** :

$$\mathrm{CD} = q_\alpha \sqrt{\frac{k(k+1)}{6N}}$$

Deux modèles sont **significativement différents** si et seulement si :

$$|\bar{r}_i - \bar{r}_j| \geq \mathrm{CD}$$

### Intuition

```
Rang moyen  1.0 ──────── 2.0 ──────── 3.0
             Hybrid     SVD++     ItemBased

CD = 1.91

|Hybrid - ItemBased| = 2.0 ≥ 1.91  ✓  SIGNIFICATIF
|Hybrid - SVD++|     = 1.0 < 1.91  ✗  pas significatif
|SVD++ - ItemBased|  = 1.0 < 1.91  ✗  pas significatif
```

### Vos résultats cross-family

- **Hybrid vs ItemBased** : $p = 0.038$ → **significativement meilleur** ✓
- Hybrid vs SVD++ : $p = 0.44$ → pas significatif (manque de puissance avec 3 folds)
- SVD++ vs ItemBased : $p = 0.44$ → pas significatif

---

## 3. Le Test de Wilcoxon + Holm-Bonferroni

### Wilcoxon — plus puissant que Nemenyi

Nemenyi utilise seulement les **rangs**. Wilcoxon utilise aussi l'**amplitude** des différences fold par fold :

```
         Fold 1   Fold 2   Fold 3   Fold 4   Fold 5
Hybrid   0.728    0.731    0.726    0.733    0.729
SVD++    0.787    0.795    0.784    0.792    0.788

Différences : 0.059  0.064  0.058  0.059  0.059
             → toutes positives, amplitude similaire
```

Il teste si ces différences sont systématiquement dans le même sens.

### Holm-Bonferroni — correction pour tests multiples

Quand on compare $\binom{k}{2}$ paires simultanément, le risque de faux positif s'accumule. Holm-Bonferroni ajuste les seuils :

```
Paires triées par p-value croissante :
  paire 1 : p = 0.002  →  testé à α/k
  paire 2 : p = 0.015  →  testé à α/(k-1)
  paire 3 : p = 0.043  →  testé à α/(k-2)
  ...
```

### Vos résultats

Avec $N = 5$ folds, le p-value minimum atteignable est $0.0625 > 0.05$ → **aucune paire n'atteint la significativité après correction**. C'est une **limitation de puissance**, pas une preuve d'équivalence.

---

## Résumé visuel de la logique

```
         FRIEDMAN
        "Y a-t-il au moins une différence ?"
              │
        p < 0.05 ?
         OUI ↓
         NEMENYI
    "Quelles paires sont différentes ?"
    │bar(r)_i - bar(r)_j│ ≥ CD ?
              │
         OUI → significatif
         NON → différence réelle mais
               pas assez de folds
               pour le prouver
```

---

## Ce qu'il faut retenir de vos tests

1. **Le hybrid est significativement meilleur que item-based CF** ($p = 0.038$) — prouvé
2. **Tuning des hyperparamètres > choix du modèle** dans la famille latente — les gains SVD tuné vs default sont significatifs ($p = 0.023$)
3. **Genome full >> genre one-hot** dans la famille contenu — significatif ($p = 0.001$)
4. **Hybrid vs SVD++** — différence réelle (0.058 RMSE) mais pas assez de folds pour le prouver statistiquement → **limitation de puissance**, pas d'équivalence
why ridge : |   |   |   |
|---|---|---|
|Intercept only|No content features used|Ridge learns per-user taste weights|
|OLS|Overfits, near-singular matrix|L2 term stabilises inversion|
|RidgeCV|No benefit, slower|Fixed α=20 already optimal globally|
|Lasso|Zeros correlated genome tags|L2 preserves all dense perceptual signal|
|Random/popularity|Ignores user preferences|Ridge personalises per use|
_"Ridge is preferred over OLS (unstable under correlated high-dimensional features), Lasso (spurious sparsity on jointly informative genome tags), and RidgeCV (empirically equivalent but slower, Table~\ref{tab:hybrid_variants}); the closed-form solution also enables real-time per-request inference in the webapp."_


,rmse, mae

ib_msd_k40_knn,0.8247290318147782, 0.6268182474520738

knn_pearson_k80,0.8307780739834031, 0.6315193357631657

cbs_genome_full_ridge_a20,0.7375454079705686, 0.5568047508143213

svdpp_f150,0.7951246724893408, 0.6037611917522222

hybrid_a02,0.7339403267191434, 0.5551559832807738