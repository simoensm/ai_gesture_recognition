---
title: "Random Forests"
authors: ["Breiman, Leo"]
year: 2001
citekey: "Breiman2001_RandomForest"
doi: "10.1023/A:1010933404324"
venue: "Machine Learning, 45(1), 5–32"
type: article
tags: [literature, random-forest, ensemble, decision-trees, bagging, feature-sampling, classification, regression]
status: read
relevance: high
created: 2026-05-22
---

# Random Forests

> [!info] Metadata
> **Authors:** Leo Breiman (UC Berkeley)
> **Year:** 2001
> **Venue:** Machine Learning, 45(1), 5–32

---

## Abstract

Introduces **Random Forests** — an ensemble method that builds many decision trees via bootstrap aggregation (bagging) and random feature subsets. Combining hundreds of decorrelated trees produces a model with low variance and robust out-of-bag error estimation.

---

## Key Contributions

- **Bagging:** Each tree is trained on a bootstrap sample (sampling with replacement) of the training data
- **Random feature subsets:** At each split, only a random subset of $m \approx \sqrt{p}$ features (for classification) or $m \approx p/3$ (for regression) are considered — decorrelates trees
- **Aggregation:** Final prediction = majority vote (classification) or mean (regression) across all trees
- **Out-of-bag (OOB) error:** Each tree was not trained on ~37% of samples → free internal cross-validation estimate
- **Feature importance:** Average decrease in impurity (Gini/entropy) from splits on each feature

## Bias-Variance Decomposition

$$\text{Error}(\hat{f}) = \text{Bias}^2 + \frac{\bar{\rho}\sigma^2}{B} + \sigma^2_\varepsilon$$

where $\bar{\rho}$ is the average pairwise correlation between trees and $B$ is the number of trees. Random feature sampling reduces $\bar{\rho}$ → reduces variance while keeping bias low.

## Hyperparameters

- `n_estimators`: number of trees (more = better, diminishing returns)
- `max_features`: $m$ features per split (key for decorrelation)
- `max_depth`: controls individual tree complexity
- `min_samples_leaf`: prevents overfitting to small splits

## Strengths and Limitations

**Strengths:**
- Handles mixed data types, missing values, high-dimensionality
- Out-of-bag error without a validation set
- Feature importance ranking
- Parallelisable

**Limitations:**
- Large memory footprint (many full trees)
- Less interpretable than a single tree
- Slower than single tree for inference (forests of 100-1000 trees)

## Connections

- Decision trees → [[Concepts/Decision_Trees_Ensembles|Decision Trees & Ensembles]]
- Boosting alternative → [[Papers/Friedman2001_GBM|Gradient Boosting (Friedman 2001)]]
- Bias-variance → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Feature importance → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]
