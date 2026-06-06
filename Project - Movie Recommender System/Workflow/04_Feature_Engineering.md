# Feature Engineering

All item feature representations used in content-based models. Each produces a feature matrix where **rows = movies** and **columns = features**.

For code details, see [[Codes/02_Feature_Engineering_Code]].

---

## The Central Function

All features are built by `build_content_features(features_method, n_genome_components=150)` in `models.py`. Pass a method name → get a pandas DataFrame indexed by movieId.

---

## Feature Families

### 🔵 Basic Metadata (Weak signal — RMSE 0.89–0.93)

| Method | Dims | Description |
|---|---|---|
| `genres` | 19 | Binary one-hot: 1 if the movie has that genre |
| `all_basic` | 20 | genres + release year, both StandardScaled |
| `release_year` | 1 | Year extracted from title string "Movie (1995)" |
| `n_genres` | 1 | Number of genres for the movie |
| `genres_tfidf` | ≤19 | TF-IDF on genre strings — rarer genres (Film-Noir) get higher weight |

**Why these are weak:** Genre labels are too coarse. Two films can share identical genre vectors (Action, Sci-Fi) yet appeal to very different users.

---

### 🟣 Visual Features (Weak — RMSE 0.93)

| Method | Dims | Description |
|---|---|---|
| `visuals` | 7 | Poster-derived colour/texture descriptors, StandardScaled |

**Why visuals don't help:** Visual aesthetics don't predict taste. A dark thriller and dark drama share near-identical poster statistics. Additionally, 47% of movies lack a poster → mean-imputed.

---

### 🟡 TMDB Text & Metadata (Moderate — RMSE 0.89–0.93)

| Method | Dims | Description |
|---|---|---|
| `tmdb_synopsis_tfidf` | 500 | TF-IDF on movie synopsis (min_df=3, log-TF) |
| `tmdb_text_tfidf` | 800 | Synopsis + tagline + genre names concatenated, TF-IDF |
| `tmdb_director` | 151 | One-hot: top 150 directors + "other" bucket |
| `tmdb_cast` | 151 | One-hot: top 150 actors + "other" bucket |
| `tmdb_cast_tfidf` | 300 | TF-IDF on actor names (downweights megastars) |
| `tmdb_svd` | n | Synopsis TF-IDF compressed to n dims via TruncatedSVD |
| `tmdb_full` | ~671 | Synopsis TF-IDF (500) + director binary (151) + runtime (1) + genres (≤19) |
| `tmdb_rating` | 1 | TMDB community rating (1–10), MinMaxScaled |

**Best TMDB result: `tmdb_full` at RMSE 0.888** — still 0.15 RMSE behind genome. Factual attributes correlate with content categories, not with taste.

---

### 🟢 Genome Features (Strong — RMSE 0.73–0.74) ⭐

| Method | Dims | Description |
|---|---|---|
| `genome_full` | 1128 | All raw genome relevance scores, mean-imputed |
| `genome` | n | TruncatedSVD compression to n components (default 150) |
| `genome_300` | 300 | SVD to 300 components |

> **Important insight:** Compressing genome via SVD actually *hurts* RMSE by ~0.007. Ridge regression inherently regularises high-dimensional inputs, so the full 1,128-dim vector is better left uncompressed. SVD introduces a lossy approximation that discards useful tag dimensions.

**Why genome is dominant:** Genome tags are *perceptual* — crowd-sourced from actual viewers. Tags like `thought-provoking`, `dark`, `plot twist` directly encode the qualities users form preferences about. This pre-processes movies into the exact space where taste lives.

---

### 🔴 Combined & Hybrid Features

| Method | Description |
|---|---|
| `combined` | Genome SVD + release year + genre one-hot, all StandardScaled |
| `genome_stats` | genome_full + 5 item quality statistics (from full dataset — leaky outside CV) |
| `genre_genome` | Genre one-hot + genome_full, both naturally in [0,1] |
| `genre_genome_weighted` | Genre one-hot ×30 + genome_full. Genre is scaled up so it can override genome for genre-homogeneous users |
| `genome_plus_rating` | genome_full + TMDB community rating |
| `genome_svd_plus` | genome SVD + visuals + item stats |
| `cf_svd` | Item latent factors extracted from the rating matrix via TruncatedSVD |
| `cf_svd_genome` | CF latent factors (n/2 dims) + genome SVD (n/2 dims), both StandardScaled |
| `cf_genome_full` | CF SVD item factors + full genome (1128 dims), both StandardScaled |

---

## Item Quality Statistics (Used in `ContentBasedWithStats`)

These are computed from the **training fold only** (no leakage) and appended to the content features:

| Statistic | Formula | What it captures |
|---|---|---|
| `item_bayesian_mean` | (n·μᵢ + 25·μ_global) / (n + 25) | Shrinkage-corrected quality (avoids noise from low-count items) |
| `item_log_count` | log(1 + n_ratings) | Popularity proxy, log-scaled |
| `item_std` | std(ratings) | Polarising (high std) vs. consensus (low std) |
| `item_high_ratio` | P(r ≥ 4) | "Loved it" fraction — orthogonal to the mean |

All four are MinMaxScaled to [0, 1] before appending. See [[Codes/06_Models_Content_Based]].

---

## TF-IDF Explained Simply

**TF-IDF** (Term Frequency–Inverse Document Frequency) turns text into numbers. For genres:
- A common genre like "Drama" appears in 40% of movies → low IDF weight
- A rare genre like "Film-Noir" appears in 1% → high IDF weight
- Result: rare, distinctive genre tags get higher representation

For synopsis text, `sublinear_tf=True` applies log(1+TF) so that a word appearing 100 times isn't 100× more important than one appearing once.

---

> [!WARNING]+ ❓ Assessor Question — Why scale genres ×30 in `genre_genome_weighted`?
> With 1,128 genome dimensions vs. 19 genre dimensions, Ridge regression's L2 penalty collectively gives the genome block 59× more "budget" to fit targets. Genre features end up with negligible individual weights and are ignored. Scaling genres ×30 approximately equalises the per-feature L2 budget between the 19-dim genre block and the 1,128-dim genome block. This lets genre explicitly dominate for users whose preferences are genre-homogeneous (e.g., a user who only watches animations).

> [!WARNING]+ ❓ Assessor Question — Why does genome SVD (compressed) perform worse than genome full?
> TruncatedSVD discards the smallest singular directions — but those directions may contain genuine user-preference signal even if they're small in the *content* space. Ridge regression already handles high-dimensional inputs through L2 regularisation, effectively zeroing out unhelpful dimensions at fit time. Pre-compressing with SVD removes potentially useful dimensions that Ridge would have kept (with near-zero weights) or discarded (by shrinking them to zero). The net effect is a lossy approximation that hurts by ~0.007 RMSE.

> [!WARNING]+ ❓ Assessor Question — What is the Bayesian mean and why use it?
> The Bayesian mean is a weighted average between the item's own mean rating and the global mean: (n·μᵢ + k·μ_global) / (n + k). With k=25 (our prior strength), a movie with only 2 ratings is pulled strongly toward the global mean — its 2-rating estimate is unreliable. A movie with 500 ratings stays close to its own mean. This prevents obscure movies with a single 5-star rating from appearing as the highest-quality item in the dataset.

---

**See also:** [[Codes/02_Feature_Engineering_Code]] | [[Codes/06_Models_Content_Based]] | [[Workflow/08_Results_and_Decision_Making]]
