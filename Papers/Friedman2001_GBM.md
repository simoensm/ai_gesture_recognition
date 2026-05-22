---
title: "Greedy Function Approximation: A Gradient Boosting Machine"
authors: ["Friedman, Jerome H."]
year: 2001
citekey: "Friedman2001_GBM"
doi: "10.1214/aos/1013203451"
venue: "Annals of Statistics, 29(5), 1189–1232"
type: article
tags: [literature, gradient-boosting, ensemble, decision-trees, boosting, gbm, xgboost, functional-gradient]
status: read
relevance: high
created: 2026-05-22
---

# Greedy Function Approximation: A Gradient Boosting Machine

> [!info] Metadata
> **Authors:** Jerome H. Friedman (Stanford)
> **Year:** 2001
> **Venue:** Annals of Statistics, 29(5), 1189–1232

---

## Abstract

Formulates boosting as **stagewise functional gradient descent** in function space. Each new tree fits the negative gradient (pseudo-residuals) of the loss, additively correcting the existing ensemble. This framework unifies boosting with gradient descent and generalises to arbitrary differentiable loss functions.

---

## Key Contributions

- **Gradient Boosting as Gradient Descent:** The ensemble $F_m(x) = F_{m-1}(x) + \nu h_m(x)$ minimises the loss by following the steepest gradient in function space
- **Pseudo-residuals:** At step $m$, fit a tree $h_m$ to:
$$r_{im} = -\left[\frac{\partial L(y_i, F(x_i))}{\partial F(x_i)}\right]_{F=F_{m-1}}$$
- Works for any differentiable loss: squared error → regression, log-loss → classification, absolute error → robust regression
- **Shrinkage (learning rate):** $\nu \in (0,1]$ shrinks each tree's contribution — reduces overfitting
- **Subsampling:** Random row sampling (stochastic GBM) further reduces variance and speeds training

## Algorithm Summary

1. Initialise $F_0(x) = \arg\min_\gamma \sum_i L(y_i, \gamma)$
2. For $m = 1, \ldots, M$:
   a. Compute pseudo-residuals $r_{im}$
   b. Fit a regression tree $h_m$ to $\{r_{im}\}$
   c. Find step size: $\gamma_m = \arg\min_\gamma \sum_i L(y_i, F_{m-1}(x_i) + \gamma h_m(x_i))$
   d. Update: $F_m(x) = F_{m-1}(x) + \nu \gamma_m h_m(x)$
3. Return $F_M(x)$

## Modern Extensions

- **XGBoost** (Chen & Guestrin, 2016): second-order Taylor expansion, regularised objective, efficient split finding
- **LightGBM** (Ke et al., 2017): histogram-based splits, leaf-wise tree growth, GOSS
- **CatBoost**: ordered boosting to prevent target leakage

## Connections

- Decision trees → [[Concepts/Decision_Trees_Ensembles|Decision Trees & Ensembles]]
- Bagging alternative → [[Papers/Breiman2001_RandomForest|Random Forests (Breiman 2001)]]
- Gradient descent → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- Regularisation → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
