---
title: "Decision Trees, Random Forests & Gradient Boosting"
tags: [concept, decision-trees, random-forest, gradient-boosting, xgboost, ensemble]
created: 2026-05-22
---

# Decision Trees, Random Forests & Gradient Boosting

> [!abstract] Powerful non-parametric models
> Decision trees split data by feature thresholds. Ensembles (Random Forest, Gradient Boosting) combine many trees to dramatically reduce variance or bias. XGBoost/LightGBM are state-of-the-art for tabular data.

**Key papers:** [[Papers/Breiman2001_RandomForest|Breiman 2001 (RF)]], [[Papers/Friedman2001_GBM|Friedman 2001 (GBM)]]

---

## 1. Decision Tree

A recursive binary partition of the feature space.

**Splitting criterion:**
- **Information Gain (ID3/C4.5):** $\text{IG}(D, A) = H(D) - \sum_v \frac{|D_v|}{|D|} H(D_v)$
- **Gini impurity (CART):** $G = 1 - \sum_k p_k^2$
- **Variance reduction (regression)**

**Greedy algorithm:** At each node, find the split $(A, t)$ that maximises information gain.

**Complexity:** $\mathcal{O}(d \cdot n \log n)$ to train a full tree.

**Prone to overfitting:** A fully-grown tree memorises training data. Prune or limit depth.

---

## 2. Random Forest (Breiman, 2001)

Train $T$ decision trees, each on:
1. A **bootstrap sample** of the training data (bagging)
2. Only a random subset of $\sqrt{d}$ features at each split

**Prediction:** majority vote (classification) or average (regression).

**Why it works:**
- Bagging reduces **variance** without increasing bias
- Feature randomness **decorrelates** trees → more benefit from averaging

**Out-of-bag (OOB) error:** Free validation estimate from unused bootstrap samples.

---

## 3. Gradient Boosting (Friedman, 2001)

Train trees **sequentially**: each tree corrects the residual errors of the previous ensemble.

**Algorithm:**
1. Initialise $F_0(x) = \arg\min_c \sum_i \ell(y_i, c)$
2. For $m = 1, \ldots, M$:
   - Compute pseudo-residuals: $r_{im} = -\left[\frac{\partial \ell(y_i, F(x_i))}{\partial F(x_i)}\right]_{F=F_{m-1}}$
   - Fit a regression tree $h_m$ to residuals $\{r_{im}\}$
   - Update: $F_m(x) = F_{m-1}(x) + \eta h_m(x)$

**Why it works:** Each tree takes a gradient step in function space.

### XGBoost / LightGBM / CatBoost

Modern implementations with:
- Regularised objective ($L_1, L_2$ on leaf weights)
- Histogram-based splits (fast)
- Parallel/distributed training
- Handling of missing values

**XGBoost** is often the first thing to try on tabular data competitions.

---

## 4. Comparison

| | Decision Tree | Random Forest | Gradient Boosting |
|---|---|---|---|
| **Bias** | Low (deep) | Low | Low |
| **Variance** | High | Low | Medium |
| **Speed** | Fast | Medium | Slower |
| **Parallelism** | — | Easy | Sequential |
| **Interpretability** | High | Medium (feature importance) | Medium |
| **Best for** | Prototype, rules | Robust baseline | Competitions, tabular SOTA |

---

## Connections

- Information gain → [[Foundations/Information_Theory|Mutual information / entropy]]
- Bias-variance tradeoff → [[Concepts/Bias_Variance_Regularisation|Regularisation]]
- Feature importance from RF → [[Foundations/Information_Theory|Mutual information]]
- Gradient boosting uses → [[Foundations/Optimisation_Theory|gradient descent in function space]]
- Alternative to deep learning for tabular data
