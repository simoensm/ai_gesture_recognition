# Configs — WebappConfig & EvalConfig

**File:** `recsys/configs.py`

These two classes are the **single control panel** for the entire project. Nothing else needs to change when you want to switch models or add an ablation variant.

---

## `WebappConfig` — Runtime Model Selector

Controls which model the Flask webapp (`server.py`) loads at startup.

### Active Setting (default)

```python
class WebappConfig:
    MODEL_CLASS        = "HybridSVD"
    ALPHA_SVD          = 0.2          # SVD weight (0=pure content, 1=pure SVD)
    N_FACTORS          = 100
    N_EPOCHS           = 30
    FEATURES_METHOD    = "genome_full"
    REGRESSOR_METHOD   = "ridge"
    RIDGE_ALPHA        = 20
    MODEL_NAME         = "Hybrid · SVD + Genome Ridge"
    MODEL_DESCRIPTION  = "Linear blend: 20% SVD + 80% per-user Ridge on 1,128 genome tags..."
```

The server reads `WebappConfig.MODEL_CLASS` before any other line executes. Based on this string, it branches into different startup code (SVD training, feature loading, etc.).

### Switching Models

All alternative model configs are kept as **commented-out blocks** below the active one. To switch, comment the active block and uncomment another:

```python
# ── (a) User-based CF ────────────────────
# MODEL_CLASS       = "UserBased"
# K_NEIGHBORS       = 40
# MODEL_NAME        = "User-Based CF · Pearson"

# ── (a-alt) Item-based CF ────────────────
# MODEL_CLASS       = "ItemBased"
# K_NEIGHBORS       = 20
# N_FACTORS         = 100

# ── (b) Content-based ────────────────────
# MODEL_CLASS       = "ContentBasedWithStats"
# FEATURES_METHOD   = "genome_full"
# REGRESSOR_METHOD  = "ridge"
# RIDGE_ALPHA       = 20

# ── (c) SVD ──────────────────────────────
# MODEL_CLASS       = "SVD"
# N_FACTORS         = 100
# N_EPOCHS          = 30
```

### Model-Specific Parameters Table

| MODEL_CLASS | Key params read by server.py |
|---|---|
| `UserBased` | `K_NEIGHBORS` |
| `ItemBased` | `K_NEIGHBORS`, `N_FACTORS` |
| `ContentBased` / `ContentBasedWithStats` | `FEATURES_METHOD`, `REGRESSOR_METHOD`, `RIDGE_ALPHA` |
| `SVD` | `N_FACTORS`, `N_EPOCHS` |
| `HybridSVD` | `ALPHA_SVD`, `N_FACTORS`, `N_EPOCHS`, `FEATURES_METHOD`, `RIDGE_ALPHA` |

---

## `EvalConfig` — Ablation Registry

Central registry of all (model_name, model_class, kwargs) triples used in `ablation_study.py`.

### Structure

```python
class EvalConfig:
    models = [
        ("model_name",  ModelClass,  {"kwarg1": val, ...}),
        ...
    ]
    split_metrics = ["rmse", "mae"]
    test_size     = 0.25
    top_n_value   = 40
```

Each tuple: `(name, class, kwargs_dict)`.

- `name` → row label in all output CSVs (`master_results.csv`, `all_results.csv`)
- `class` → the Python class to instantiate from `models.py`
- `kwargs` → passed to `ModelClass(**kwargs)` — must match the constructor signature

### How ablation_study.py Uses It

```python
for name, cls, params in EvalConfig.models:
    model = cls(**params)
    kf = KFold(n_splits=n_folds)
    for trainset, testset in kf.split(data):
        model.fit(trainset)
        preds = model.test(testset)
        fold_rmse = accuracy.rmse(preds)
```

The loop is the same for all models. Adding a new model = adding one line to `EvalConfig.models`.

### Active (Default) Models

The 5 active models (uncommented) represent the **best per family** for the final comparison:

```python
models = [
    # Best item-based CF
    ("ib_msd_k40_knn",  KNNWithMeans,  {"k": 40, "sim_options": {"name": "msd", "user_based": False, "min_support": 3}}),
    # CV RMSE ≈ 0.8371

    # Best user-based CF
    ("knn_pearson_k80", KNNWithMeans,  {"k": 80, "sim_options": {"name": "pearson", "user_based": True, "min_support": 3}}),
    # CV RMSE = 0.8280 ±0.0023

    # Best content-based
    ("cbs_genome_full_ridge_a20", ContentBasedWithStats, {"features_method": "genome_full", "regressor_method": "ridge", "alpha": 20}),
    # CV RMSE = 0.7344 ±0.0016

    # Best latent factor
    ("svdpp_f150",      SVDppModel,    {"n_factors": 150, "n_epochs": 30, "lr_all": 0.010, "reg_all": 0.05}),
    # CV RMSE = 0.7890 ±0.0024

    # Best overall
    ("hybrid_a02",      HybridSVD,    {"alpha": 0.2, "features_method": "genome_full", "regressor_method": "ridge",
                                        "content_alpha": 20, "n_factors": 100, "reg_all": 0.05, "lr_all": 0.010, "n_epochs": 30}),
    # CV RMSE = 0.7308
]
```

### Commented Sweep Entries

The commented lines document every variant tested during the ablation. Each has its CV RMSE annotated inline:

```python
#("knn_cosine_k20",  KNNWithMeans, {...}),  # CV RMSE=0.8422 ±0.0002
#("knn_cosine_k40",  KNNWithMeans, {...}),  # CV RMSE=0.8402 ±0.0005
...
#("ub_spearman_k40_ms1", UserBased, {...}), # CV RMSE=0.8335 ±0.0006 ← best UserBased
```

This is the project's **experiment log** — every hyperparameter configuration tried is documented here with its result.

### Naming Convention

`{family}_{key_params}_{settings}`:
- `knn_` → Surprise KNNWithMeans
- `ub_` → custom UserBased
- `ib_` → custom ItemBased or KNNWithMeans item-based
- `cbs_` → ContentBasedWithStats
- `svd_` / `svdpp_` → latent factor
- `hybrid_` → HybridSVD
- `_a02` → alpha=0.2
- `_k40` → k=40 neighbors
- `_f100` → 100 factors
- `_ms3` → min_support=3

---

> [!WARNING]+ ❓ Assessor Question — Why keep all the commented entries instead of deleting them?
> The commented entries serve as the project's **experiment log**. They document every configuration tested, with CV RMSE annotated inline. If you need to reproduce a specific result (e.g., to answer "what was the effect of changing k from 20 to 40?"), you can uncomment the relevant lines and re-run the ablation. Deleting them would lose this historical record. The comments are documentation, not dead code.

> [!WARNING]+ ❓ Assessor Question — How does `server.py` know about `WebappConfig` at runtime if the user changes `MODEL_CLASS` interactively?
> `server.py` first calls `_select_model()` to get the user's choice (printed menu, terminal input), then immediately writes it back: `WebappConfig.MODEL_CLASS = _chosen_class`. This mutation happens **before** any model-loading code executes (the `if model_class == "UserBased":` branches). So the config change takes effect for the entire startup sequence. All further reads of `WebappConfig.MODEL_CLASS` see the user's choice, not the default in the file.

---

**See also:** [[Codes/06_Models_Content_Based]] | [[Codes/07_Models_Hybrid]] | [[Webapp/01_Webapp_Overview]] | [[Workflow/06_Ablation_Study]]
