---
title: "Mathematical & Statistical Foundations — MOC"
tags: [MOC, foundations, mathematics, statistics, probability]
created: 2026-05-22
---

# Mathematical & Statistical Foundations — MOC

> The bedrock that all AI/ML methods sit on. Every algorithm in [[Papers/00_Papers_MOC|the Papers folder]] ultimately reduces to ideas here.

---

## 🎲 Probability & Statistics

| Note | Core idea |
|---|---|
| [[Probability_Theory]] | Sample spaces, random variables, distributions, expectation |
| [[Bayesian_Inference]] | Prior → likelihood → posterior; Bayes' theorem; conjugate priors |
| [[Maximum_Likelihood_Estimation]] | MLE, MAP, connection to loss functions |
| [[Statistical_Learning_Theory]] | VC dimension, PAC learning, generalisation bounds, bias-variance |

---

## 📐 Linear Algebra & Geometry

| Note | Core idea |
|---|---|
| [[Linear_Algebra_for_ML]] | Vectors, matrices, eigendecomposition, SVD, PCA geometry |

---

## 📡 Information Theory

| Note | Core idea |
|---|---|
| [[Information_Theory]] | Entropy, KL divergence, mutual information, cross-entropy loss |

---

## ⚙️ Optimisation

| Note | Core idea |
|---|---|
| [[Optimisation_Theory]] | Convexity, gradient descent, SGD, Adam, learning rate schedules, convergence |

---

## 🔗 Connections to the Rest of the Vault

- Every loss function in ML is derived from → [[Maximum_Likelihood_Estimation]] or [[Information_Theory]]
- Generalisation theory → [[Statistical_Learning_Theory]]
- Gradient-based learning → [[Optimisation_Theory]]
- Dimensionality reduction → [[Linear_Algebra_for_ML]]
- All classification algorithms → [[Bayesian_Inference]] or [[Statistical_Learning_Theory]]
- Generative models (VAE) → [[Bayesian_Inference]] + [[Information_Theory]]
