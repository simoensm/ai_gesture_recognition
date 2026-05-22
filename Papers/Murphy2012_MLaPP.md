---
title: "Machine Learning: A Probabilistic Perspective"
authors: ["Murphy, Kevin P."]
year: 2012
citekey: "Murphy2012_MLaPP"
doi: "10.7551/mitpress/9780262018029.001.0001"
venue: "MIT Press"
type: book
tags: [literature, textbook, supervised-learning, unsupervised-learning, bayesian, probabilistic, graphical-models, deep-learning, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Machine Learning: A Probabilistic Perspective (MLaPP)

> [!info] Metadata
> **Authors:** Kevin P. Murphy (Google Brain / DeepMind)
> **Year:** 2012
> **Publisher:** MIT Press
> **Successor:** *Probabilistic Machine Learning: Introduction* (Murphy, 2022) — free at [probml.ai](https://probml.github.io/pml-book/)

---

## Abstract

A comprehensive, modern treatment of machine learning from a unified probabilistic perspective. Murphy covers more algorithms than either PRML or ESL, with clean notation and Python code examples. The 2022 successor (*Probabilistic Machine Learning: Introduction*) updates everything and is freely available online.

---

## Chapter Map (Key Chapters)

| Chapter | Topic | Highlights |
|---|---|---|
| 1–2 | **Introduction & Probability** | Review of probability, Bayesian inference, decision theory |
| 3 | **Generative Models for Discrete Data** | Naive Bayes, language models, mixture models |
| 4 | **Gaussian Models** | MVN, linear Gaussian systems, MLE for Gaussian |
| 5 | **Bayesian Statistics** | Conjugate priors, hierarchical models, posterior summaries |
| 6 | **Frequentist Statistics** | MLE, bootstrap, hypothesis testing |
| 7 | **Linear Regression** | OLS, ridge, lasso, robust regression, GP regression |
| 8 | **Logistic Regression** | Gradient methods, IRLS, multi-class, Bayesian logistic |
| 9 | **Generative vs Discriminative** | Naive Bayes vs. logistic, LDA vs. logistic |
| 10 | **Directed Graphical Models** | Bayes nets, d-separation, plate notation |
| 11 | **Mixture Models and EM** | k-means, GMM, EM convergence proof |
| 12 | **Latent Linear Models** | PCA, probabilistic PCA, factor analysis, ICA |
| 13 | **Sparse Linear Models** | Lasso, Bayesian variable selection, automatic relevance determination |
| 14 | **Kernels** | Mercer kernels, Gaussian processes, SVM |
| 16 | **Adaptive Basis Functions** | Neural networks, boosting |
| 17 | **Markov Models** | HMMs, Viterbi, Baum-Welch |
| 18–20 | **State Space Models** | Kalman filter, particle filter, switching models |
| 24 | **MCMC** | Metropolis-Hastings, Gibbs, HMC |
| 25 | **Variational Inference** | Mean-field, ELBO, stochastic VI |
| 27 | **Deep Learning** | CNNs, RNNs, autoencoders, DBMs |
| 28 | **Clustering** | k-means, spectral, affinity propagation |

---

## Why MLaPP is Valuable

- **Breadth:** Covers every major ML algorithm — supervised, unsupervised, sequential, Bayesian, frequentist
- **Probabilistic unification:** Every model is a probabilistic graphical model; every algorithm is inference
- **Code:** MATLAB/Python code available for most algorithms
- **2022 update:** *Probabilistic Machine Learning: Introduction* and *Advanced Topics* are freely available and fully updated with deep learning

---

## Key Reference Points for the Course

- **Ch. 8:** Logistic regression — clean derivation including multi-class softmax
- **Ch. 9:** Generative vs. discriminative comparison (Ng & Jordan analysis)
- **Ch. 11:** EM algorithm — proof that EM never decreases likelihood
- **Ch. 17:** HMMs — forward-backward, Viterbi, Baum-Welch
- **Ch. 16.5:** Backpropagation — cleaner notation than PRML

---

## Connections

- Logistic regression → [[Concepts/Logistic_Regression|Logistic Regression]]
- Naive Bayes → [[Concepts/Probabilistic_Classifiers|Probabilistic Classifiers]]
- EM algorithm → [[Concepts/Clustering_EM|Clustering & EM Algorithm]]
- HMMs → [[Papers/Fink2014_MarkovModels|Fink 2014]]
- PCA → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]
- Lasso → [[Papers/Tibshirani1996_Lasso|Tibshirani 1996]]
- Companions → [[Papers/Bishop2006_PRML|Bishop 2006 (PRML)]], [[Papers/Hastie2009_ESL|Hastie 2009 (ESL)]]
