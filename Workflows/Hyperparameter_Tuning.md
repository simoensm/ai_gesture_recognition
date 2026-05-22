---
title: "Hyperparameter Tuning"
tags: [workflow, hyperparameter-tuning, grid-search, random-search, bayesian-optimisation, practical]
created: 2026-05-22
---

# Hyperparameter Tuning

> Systematic strategies for finding the best model configuration. Hyperparameters are not learned from data during training — they must be set before or searched over externally.

---

## What Are Hyperparameters?

| Category | Examples |
|---|---|
| **Architecture** | Number of layers, hidden units, kernel sizes, attention heads |
| **Optimisation** | Learning rate, batch size, momentum $\beta_1$, $\beta_2$ |
| **Regularisation** | Dropout rate, L2 weight decay, early stopping patience |
| **Preprocessing** | Sequence length $T$, number of codebook clusters $K$ |
| **Model-specific** | SVM $C$ and $\gamma$, RF `n_estimators` and `max_features`, DTW band width |

---

## Why Not Tune on the Test Set?

Tuning on the test set is **data snooping** — it inflates reported performance and the model has effectively been trained on the test data. Always tune on a **validation set** (or validation folds within a CV loop).

---

## Strategy 1: Grid Search

Exhaustively evaluate all combinations in a predefined grid.

```python
param_grid = {
    'learning_rate': [1e-4, 1e-3, 1e-2],
    'dropout': [0.2, 0.3, 0.5],
    'hidden_size': [64, 128, 256]
}
# Total: 3 × 3 × 3 = 27 evaluations
```

- **Pros:** Reproducible, covers all points, good when each run is fast
- **Cons:** Exponential in number of hyperparameters — 5 parameters × 5 values = 3,125 runs

---

## Strategy 2: Random Search (Bergstra & Bengio, 2012)

Sample hyperparameter combinations uniformly at random for a fixed budget $n$ evaluations.

```python
for _ in range(n_trials):
    lr = 10 ** np.random.uniform(-4, -1)   # log-uniform
    dropout = np.random.uniform(0.1, 0.6)
    hidden = np.random.choice([64, 128, 256, 512])
    evaluate(lr, dropout, hidden)
```

**Key insight:** If only 2 of 5 hyperparameters really matter, random search explores far more unique values of the important ones than grid search does for the same number of evaluations.

- **Pros:** Often better than grid search at the same budget; easily parallelisable
- **Cons:** No guarantee of finding the optimum; no learning from previous runs

---

## Strategy 3: Bayesian Optimisation

Builds a **surrogate model** (typically Gaussian Process or Tree Parzen Estimator) of the objective function $f(\lambda)$, uses it to choose the next hyperparameter configuration that balances exploration and exploitation.

### Acquisition Functions
- **Expected Improvement (EI):** $\text{EI}(\lambda) = \mathbb{E}[\max(f(\lambda) - f^*, 0)]$ — prioritises configurations likely to beat the current best
- **Upper Confidence Bound (UCB):** $\mu(\lambda) + \kappa\sigma(\lambda)$ — $\kappa$ controls exploration-exploitation

### Tools
- **Optuna** — Python library, Tree-structured Parzen Estimator (TPE), easy pruning of bad trials
- **Hyperopt** — TPE-based, widely used
- **Ax / BoTorch** — Facebook's platform with Gaussian Process surrogates
- **Ray Tune** — distributed hyperparameter search

**When to use:** When each evaluation is expensive (minutes/hours of training). Bayesian BO outperforms random search when budget is limited.

---

## Strategy 4: Learning Rate Finding

Before tuning all hyperparameters, find the learning rate range:

**LR Range Test (Smith, 2017):**
1. Start with $\eta_{min} = 10^{-7}$, linearly increase to $\eta_{max} = 10^{-1}$ over one epoch
2. Plot loss vs. LR
3. Choose LR just before the loss starts to diverge — typically at the steepest descent

```
Loss
 │    ╲
 │     ╲__  ← optimal range
 │         ╲__/↗ ← diverges here
 └──────────────── Learning rate (log scale)
```

---

## Practical Tuning Protocol

### Recommended Order (most impactful first)

1. **Learning rate** — the single most important hyperparameter. Test: `[1e-4, 3e-4, 1e-3, 3e-3, 1e-2]`
2. **Batch size** — larger batches need larger LRs (linear scaling rule). Typical: 32–256
3. **Architecture width/depth** — number of layers and units
4. **Regularisation** — dropout, weight decay
5. **Other** — momentum, $\beta_1$, $\beta_2$ (rarely need to move from Adam defaults)

### Budget-aware advice

| Budget | Strategy |
|---|---|
| < 20 evaluations | Manual + intuition, or 1D LR sweep |
| 20–100 evaluations | Random search |
| 100–1000 evaluations | Bayesian optimisation (Optuna) |
| > 1000 evaluations | Population-based training (PBT) |

---

## Early Stopping

Stop training when validation loss has not improved for `patience` epochs:

```python
best_val_loss = inf
patience_counter = 0
for epoch in range(max_epochs):
    train_epoch()
    val_loss = evaluate()
    if val_loss < best_val_loss - delta:
        best_val_loss = val_loss
        save_checkpoint()
        patience_counter = 0
    else:
        patience_counter += 1
        if patience_counter >= patience:
            break  # early stop
```

- Typical patience: 10–50 epochs for deep learning
- Restore best checkpoint before reporting test performance

---

## Nested Cross-Validation

When the dataset is small and you need both hyperparameter selection and unbiased test evaluation:

```
Outer loop (evaluation): k₁ folds
    Inner loop (tuning): k₂ folds per outer fold
```

- Inner CV: select best hyperparameters
- Outer CV: estimate generalisation error of the selection procedure
- Expensive: $k_1 \times k_2 \times n_{trials}$ model evaluations

---

## Connections

- Full pipeline → [[Workflows/ML_Pipeline|ML Pipeline]]
- Learning rate theory → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- Validation protocol → [[Concepts/Cross_Validation|Cross-Validation]]
- Regularisation → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Model evaluation → [[Workflows/Model_Evaluation|Model Evaluation]]
