# Data Description

The project uses the **hackathon split** of the MovieLens-1M dataset.

---

## Core Files

| File | Location | Content |
|---|---|---|
| `ratings.csv` | `data/hackathon/evidence/` | userId, movieId, rating, timestamp |
| `movies.csv` | `data/hackathon/content/` | movieId, title (with year), genres |
| `genome-scores.csv` | `data/hackathon/content/` | movieId, tagId, relevance score |
| `genome-tags.csv` | `data/hackathon/content/` | tagId → human-readable tag name |
| `LLVisualFeatures13K_QuantileLog.csv` | `data/hackathon/content/visuals/` | 7 visual poster descriptors |

Constants for all paths are defined in [[Codes/01_Constants_and_Loaders]].

---

## Dataset Statistics

| Property | Value |
|---|---|
| Users | ~1,000 |
| Movies (total) | ~8,744 |
| Ratings | ~180,000 |
| Rating scale | {0.5, 1.0, 1.5, …, 5.0} |
| Sparsity | > 97% |
| Global mean rating | ~3.5 |
| Avg ratings per user | ~180 |

The **rating matrix** is a 1,000 × 8,744 table where most cells are empty. This extreme sparsity is why collaborative filtering struggles: most user pairs share very few co-rated movies.

---

## Item Features

### 1. Genre One-Hot (19 dimensions)
Each movie is tagged with 1–3 genres (Action, Drama, Comedy, etc.). The 19-dimensional binary vector has a 1 for each genre present. Fast to build, but genres are coarse labels that don't distinguish *taste* within a genre.

### 2. Genome Tags (1,128 dimensions) ⭐ Most powerful
The **MovieLens Tag Genome** (Vig et al. 2012) is a crowd-sourced matrix of 1,128 tags × all movies, where each cell is a relevance score in [0, 1].

Examples of tags: `thought-provoking`, `plot twist`, `visually stunning`, `cult classic`, `mind-bending`.

These are **perceptual** attributes — what viewers subjectively experience — which directly correlate with preference dimensions. This is why genome features are far more predictive than factual metadata.

> **Missing genome entries:** 365 of 8,737 movies (4.2%) have no genome vector. These are long-tail films with fewer than 5 ratings. We **mean-impute** missing rows (fill with the column mean across all movies). The RMSE impact is < 0.001.

### 3. Visual Features (7 dimensions)
7 poster-derived descriptors (brightness, contrast, colour histograms) from the LL Visual Features dataset (Deldjoo et al. 2016). Available for ~53% of movies. The rest are mean-imputed.

> **Why visuals don't help:** A dark thriller and a dark drama have nearly identical visual statistics but appeal to completely different audiences. Visual aesthetics do not encode preference.

### 4. TMDB Metadata (enriched)
Collected via the TMDB API using `scripts/enrich_all_movies.py`:

| Field | Use |
|---|---|
| `synopsis` | TF-IDF text representation |
| `director` | Binary one-hot (top 150 directors) |
| `cast` | Binary one-hot or TF-IDF (top 150 actors) |
| `runtime` | MinMax-scaled to [0,1] |
| `tmdb_rating` | Global community quality signal |
| `tagline` | Concatenated with synopsis for text features |

TMDB metadata tops out at **RMSE ≈ 0.888** — far behind genome (0.735). Factual attributes correlate with content *categories*, not with *taste*.

---

## Rating Distribution

The distribution is **right-skewed**:
- Most common ratings: 4.0 and 3.0
- Global mean: ~3.5
- ~10% of ratings are half-stars (0.5, 1.5, 2.5, …)

This positive bias motivates:
- **Per-user mean-centering** in collaborative filtering (remove rating-scale differences)
- **Bayesian shrinkage** in item quality statistics (prevent noise from low-count items)

---

## Available Data Subsets

| Subset | Purpose |
|---|---|
| `test` | Understand the data structure, quick sanity checks |
| `tiny` | Fast local iteration (seconds per run) |
| `small` | Full-scale without genome |
| `hackathon` | Full dataset with genome + visuals. **All results in this project use this.** |

To switch subset, change `DATA_PATH` in `constants.py`.

---

> [!WARNING]+ ❓ Assessor Question — Why are genome features so much more powerful than TMDB metadata?
> Genome tags are **perceptual** attributes collected from actual viewers: they encode what a movie *feels like* rather than what it *is*. A tag like `thought-provoking` or `dark` directly captures the qualities users form preferences about. TMDB synopsis TF-IDF encodes narrative words (`heist`, `space`) that correlate with genre but not with taste intensity. Two action films can both contain the word `explosion` but be received very differently by the same user. The genome essentially pre-processes the movies into the space of human perception — the exact space where preferences live.

> [!WARNING]+ ❓ Assessor Question — What is sparsity and why does it matter?
> Sparsity = (number of empty cells) / (total cells in the rating matrix). At 97% sparsity, the average user has rated only ~180 out of 8,744 movies (about 2%). This means: (1) most user pairs share very few co-rated items, making similarity estimates unreliable in CF; (2) most items have very few ratings, making item quality hard to estimate; (3) any model that operates on the raw rating matrix (CF, SVD) must handle these missing values, usually by imputing with 0 or the global mean.

> [!WARNING]+ ❓ Assessor Question — What is mean-imputation and what are its risks?
> Mean-imputation replaces a missing value with the average of that column (or row). For genome features, we fill missing movie rows with the average relevance score per tag across all movies. Risk: the imputed vector is identical for all missing movies, so the model treats them as "average" films regardless of their true content. Since only 4.2% of movies are affected (all long-tail), the RMSE impact is negligible (< 0.001).

---

**See also:** [[Codes/01_Constants_and_Loaders]] | [[Workflow/04_Feature_Engineering]] | [[Workflow/01_Project_Overview]]
