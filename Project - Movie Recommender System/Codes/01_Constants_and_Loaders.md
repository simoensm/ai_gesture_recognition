---
tags:
  - recommender-systems
  - code
  - python
  - data-loading
  - MLSMM2156
---
# Constants & Loaders

**Files:** `recsys/constants.py` and `recsys/loaders.py`

These two files are the foundation of the data pipeline. Everything else imports from them.

---

## `constants.py` — Central Path Registry

### What it does
Defines one `Constant` class with all data paths and column names as class attributes. No logic, just configuration.

### Why use a class instead of module-level variables?
A class groups everything under `C.` (imported as `from constants import Constant as C`). One import gives you access to all constants. If you need to change the dataset size (from `tiny` to `hackathon`), you change one line: `DATA_PATH = Path(__file__).parent.parent / 'data/hackathon'`.

### Key attributes

```python
# Path to the dataset root (change this to switch between tiny/small/hackathon)
DATA_PATH = Path(__file__).parent.parent / 'data/hackathon'

# Derived paths
CONTENT_PATH  = DATA_PATH / 'content'      # movies, genome, visuals
EVIDENCE_PATH = DATA_PATH / 'evidence'     # ratings.csv
VISUALS_PATH  = CONTENT_PATH / 'visuals'

# Column names (used everywhere to avoid typos)
ITEM_ID_COL  = 'movieId'
USER_ID_COL  = 'userId'
RATING_COL   = 'rating'
GENRES_COL   = 'genres'
LABEL_COL    = 'title'

# Rating scale (used by Surprise Reader)
RATINGS_SCALE = (0.5, 5.0)
```

---

## `loaders.py` — Data Loading Functions

### `load_ratings(surprise_format=False)`

```python
def load_ratings(surprise_format=False):
    df = pd.read_csv(C.EVIDENCE_PATH / C.RATINGS_FILENAME)
    if surprise_format:
        reader = Reader(rating_scale=C.RATINGS_SCALE)
        return Dataset.load_from_df(df[C.USER_ITEM_RATINGS], reader)
    return df
```

**What it does:** Loads `ratings.csv` as a pandas DataFrame OR as a Surprise `Dataset` object.

**`surprise_format=True`:** Wraps the DataFrame in Surprise's `Dataset`, which is required by all Surprise models (`KFold`, `SVD`, etc.). The `Reader` tells Surprise the rating scale so it can clip predictions correctly.

**When to use which:**
- `False` → when you need pandas operations (groupby, pivot, statistics)
- `True` → when you're about to train a Surprise model

---

### `load_items()`

```python
def load_items():
    df = pd.read_csv(C.CONTENT_PATH / C.ITEMS_FILENAME)
    return df.set_index(C.ITEM_ID_COL)
```

Returns a DataFrame indexed by `movieId` with columns `title` and `genres`. The index is the movie ID, making it easy to join with other DataFrames or look up a movie by ID.

---

### `load_genome()` — Cached

```python
@lru_cache(maxsize=1)
def load_genome():
    df = pd.read_csv(genome_path)
    return df.pivot(index='movieId', columns='tagId', values='relevance')
```

**What it does:** Loads the genome scores (9.4 million rows!) and pivots them into a movieId × tagId matrix. Each cell is the relevance score [0,1] for that (movie, tag) pair.

**`@lru_cache(maxsize=1)`**: This is a **memoization decorator**. The first call loads the file (takes ~5 seconds). Every subsequent call returns the cached result instantly. Without this, any code that calls `load_genome()` multiple times would reload 9.4 million rows each time.

**Why `pivot()`?** The raw CSV has 3 columns: `movieId`, `tagId`, `relevance`. With 9.4M rows, it's a long-format table. The pivot converts it to wide format: rows = movies, columns = tags. This is the matrix format needed for feature engineering.

---

### `load_tmdb()` — Cached

```python
@lru_cache(maxsize=1)
def load_tmdb():
    movies = json.loads(_WEBAPP_MOVIES.read_text())
    records = []
    for m in movies:
        cast_names = ' '.join(
            c['name'].replace(' ', '_') for c in (m.get('cast') or [])
        )
        records.append({
            'movieId': int(m['id']),
            'synopsis': (m.get('synopsis') or '').strip(),
            ...
        })
    return pd.DataFrame(records).set_index('movieId')
```

**Source:** `webapp/public/movies.json` — enriched by `scripts/enrich_all_movies.py` using the TMDB API.

**Why replace spaces in actor names with underscores?** `"Tom Hanks"` would be split into two tokens `"Tom"` and `"Hanks"` by whitespace tokenization. By storing it as `"Tom_Hanks"`, TF-IDF and split-on-whitespace operations correctly treat it as one actor token.

---

### `export_evaluation_report(df)`

Saves a results DataFrame to `data/hackathon/evaluations/<today_date>.csv`. Used by the evaluation notebook to version results by date.

---

> [!WARNING]+ ❓ Assessor Question — What is `lru_cache` and why is it used for genome loading?
> `lru_cache` (Least Recently Used cache) is a Python decorator that memoizes (caches) function results. With `maxsize=1`, it stores the result of the most recent call and returns it on all subsequent calls without re-executing the function. `load_genome()` reads and pivots a 9.4-million-row CSV, which takes ~5 seconds. Without caching, any code that called `load_genome()` twice would spend 10+ seconds loading data. With caching, only the first call does the work; all subsequent calls are near-instant. This is especially important in the ablation study, which calls `build_content_features()` for each of the 5 CV folds.

> [!WARNING]+ ❓ Assessor Question — Why is `set_index('movieId')` important?
> Setting `movieId` as the DataFrame index allows fast O(1) lookups by movie ID: `df.loc[movieId]` instead of filtering the entire DataFrame. More importantly, pandas operations like `.reindex()`, `.merge()`, and concatenation align DataFrames on their index automatically. When we build a feature matrix and later look up a specific movie's features, the index lookup is what makes this efficient. Without an index, we'd need to search the entire DataFrame for each movie ID.

---

**See also:** [[Workflow/02_Data]] | [[Codes/02_Feature_Engineering_Code]]
