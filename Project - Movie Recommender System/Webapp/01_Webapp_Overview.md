# Webapp Overview

**File:** `recsys/server.py`, `webapp/` (React frontend)

The webapp is a movie recommender interface backed by the best-performing model. It communicates with a Flask API server that loads the model at startup and responds to HTTP requests.

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│  Browser (React frontend)                            │
│  webapp/src/  →  served from webapp/public/          │
└──────────────────┬───────────────────────────────────┘
                   │  HTTP  (POST /recommend, POST /similar)
┌──────────────────▼───────────────────────────────────┐
│  Flask API  (recsys/server.py)                       │
│  localhost:5001                                      │
│  ┌──────────────────────────────────────────────────┐│
│  │  Model loaded at startup (single instance)       ││
│  │  WebappConfig.MODEL_CLASS = "HybridSVD"          ││
│  └──────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────┐│
│  │  Cold-start layer: genome_full always available  ││
│  └──────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────┘
```

---

## Startup Sequence

When you run `python3 server.py`:

### Step 1 — Interactive Model Selection

```
┌──────────────────────────────────────────────────────────────┐
│            RecSys MLSMM2156  —  Model Selection             │
├──────────────────────────────────────────────────────────────┤
│  1.  User-based CF       (Pearson, k=80)                    │
│  2.  Content-based       (Genome full + Ridge α=20)         │
│  3.  Latent Factor       (SVD, f=100, tuned)                │
│  4.  Hybrid              (SVD α=0.2 + Genome) ← best        │
└──────────────────────────────────────────────────────────────┘
```

The user types 1–4. `_select_model()` returns the model class name as a string, which overwrites `WebappConfig.MODEL_CLASS`.

### Step 2 — Model-Specific Startup

Depending on the chosen class, the server executes one of five startup branches:

| Choice | What happens at startup | Time |
|---|---|---|
| `UserBased` | Load ratings, build n_users × n_items pivot, compute unit user vectors | 15–30 s |
| `ItemBased` | Load ratings, TruncatedSVD(k=100) on item×user matrix, normalise | 20–40 s |
| `ContentBasedWithStats` | Load genome features + compute item stats from full ratings | 15–30 s |
| `SVD` | Train Surprise SVD on full dataset, extract Q (item factors) and b_i | 30–90 s |
| `HybridSVD` | Train SVD + load genome features + compute item stats (3 steps) | 40–90 s |

### Step 3 — Cold-Start Layer

Regardless of the main model, the server also loads genome_full features for the **cascade cold-start system**:
- If the main model is content-based or hybrid → reuse the already-loaded feature matrix (free)
- If the main model is UserBased/ItemBased/SVD → load genome_full separately (one extra I/O)

This means even a pure collaborative filtering model can handle completely new users who haven't rated anything yet.

### Step 4 — Popularity Fallback

```python
_fallback_json  = ROOT / "webapp/public/movies.json"
_fallback_order = [m["id"] for m in _all_movies]

# Pre-compute popularity score array: rank-0 → 5.0, out-of-list → 0.5
_pop_scores_arr = np.array([5.0 - 4.5 * (rank / n_pop) for mid in _cold_movie_ids])
```

A pre-ranked popularity list is always available as a fallback. Used in Phase 0/1 when preference anchors are insufficient. See [[Webapp/02_Server_Endpoints]].

---

## Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Liveness check; returns model name, feature label, n_items |
| POST | `/recommend` | Personalised recommendations (the main endpoint) |
| POST | `/similar` | Item-to-item similarity in the active feature space |

Full endpoint documentation: [[Webapp/02_Server_Endpoints]]

---

## Key Data Structures

### `_meta_map` — Movie Metadata Index

```python
_meta_map: dict[int, dict] = {
    movie_id: {
        "genres":   {"Action", "Drama", ...},
        "year":     2005,
        "director": "christopher nolan",
        "cast":     {"christian bale", "heath ledger"},
    }
}
```

Built from `webapp/public/movies.json` at startup. Used by `_prefs_to_pseudo_ratings()` to score each movie against stated user preferences. Lookup is O(1) per movie.

### `_GENOME_TAG_MAP` — Preference UI to Genome ID

```python
_GENOME_TAG_MAP = {
    "plot_twist":      789,
    "suspense":        999,
    "dark_comedy":     286,
    "mind_bending":    652,
    ...
}
```

Maps the onboarding UI's preference labels (e.g. `"plot_twist"`) to their genome tag ID. When a new user selects "plot_twist", the server looks up all movies with genome-score > 0.80 for tag 789 as cold-start anchors.

### `_genome_tags_for_movies()` — Recommendation Explanation

After fitting the Ridge regressor for a content-based prediction, this function returns the top genome tags that explain each recommendation:
- Takes the user's Ridge coefficient vector `coef` (1,128 dims for genome_full)
- For each recommended movie, finds where the user's top-weighted tags intersect with the movie's high-relevance tags
- Returns a dict `{movie_id: ["tag1", "tag2", ...]}`

This powers the "why this movie?" explanation in the frontend.

---

> [!WARNING]+ ❓ Assessor Question — Why load the model once at startup instead of once per request?
> Loading the model per-request would be catastrophically slow. Training SVD takes 30–90 seconds; loading the genome feature matrix takes 15–30 seconds. Per-request loading would mean each user waits a minute before seeing any results. By loading once at startup and keeping the model in memory as a module-level variable, each `/recommend` call only needs to run the fast inference step (milliseconds to seconds). This is standard production serving architecture.

> [!WARNING]+ ❓ Assessor Question — What does `build_full_trainset()` do vs the K-fold splitting used in evaluation?
> In evaluation (`ablation_study.py`), `KFold.split(data)` creates train/test splits for cross-validation. In the webapp, `dataset.build_full_trainset()` builds a trainset from ALL available ratings — no held-out test set. This is intentional: at serving time, we want the model to learn from all available data (more data = better embeddings). The trade-off is that we can no longer compute an honest RMSE on this model — but that's fine because the RMSE was already measured during CV evaluation.

---

**See also:** [[Webapp/02_Server_Endpoints]] | [[Codes/09_Configs]] | [[Codes/07_Models_Hybrid]]
