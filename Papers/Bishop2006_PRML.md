---
title: "Pattern Recognition and Machine Learning"
authors: ["Bishop, Christopher M."]
year: 2006
citekey: "Bishop2006_PRML"
doi: "10.1007/978-0-387-45528-0"
venue: "Springer"
type: book
tags: [literature, textbook, supervised-learning, bayesian, probabilistic, neural-networks, graphical-models, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Pattern Recognition and Machine Learning (PRML)

> [!info] Metadata
> **Authors:** Christopher M. Bishop (Microsoft Research Cambridge)
> **Year:** 2006
> **Publisher:** Springer
> **Free PDF:** [microsoft.com/en-us/research/people/cmbishop/prml-book](https://www.microsoft.com/en-us/research/people/cmbishop/prml-book/)

---

## Abstract

The definitive probabilistic treatment of machine learning. Bishop frames every algorithm — from linear regression to neural networks, SVMs, and graphical models — as a problem of probabilistic inference. Bayesian reasoning is woven through every chapter, making the mathematical connections between algorithms explicit.

---

## Chapter Map

| Chapter | Topic | Core concepts |
|---|---|---|
| 1 | **Introduction** | Polynomial regression, curse of dimensionality, probability theory review, decision theory |
| 2 | **Probability Distributions** | Gaussian, Bernoulli, Multinomial, exponential family, non-parametric methods |
| 3 | **Linear Models for Regression** | OLS, ridge, MLE, MAP, Bayesian linear regression, evidence approximation |
| 4 | **Linear Models for Classification** | Discriminant functions, logistic regression, Laplace approximation, probit model |
| 5 | **Neural Networks** | MLP, backpropagation, regularisation, Bayesian neural networks |
| 6 | **Kernel Methods** | Dual representation, constructing kernels, Gaussian processes |
| 7 | **Sparse Kernel Machines** | SVM, margin, SMO, relevance vector machine |
| 8 | **Graphical Models** | Bayesian networks, Markov random fields, factor graphs, d-separation |
| 9 | **Mixture Models and EM** | k-means, Gaussian mixture, EM algorithm, latent variable models |
| 10 | **Approximate Inference** | Variational Bayes, ELBO, expectation propagation |
| 11 | **Sampling Methods** | MCMC, Metropolis-Hastings, Gibbs sampling, HMC |
| 12 | **Continuous Latent Variables** | PCA, probabilistic PCA, factor analysis, ICA |
| 13 | **Sequential Data** | Markov models, HMMs, Kalman filter, particle filter |
| 14 | **Combining Models** | Committees, boosting, tree-based models, conditional mixture |

---

## Must-Read Sections for the Course Project

- **Ch. 3.3:** Ridge regression as MAP with Gaussian prior — connects regularisation to Bayesian inference
- **Ch. 4.3–4.4:** Logistic regression derivation and Newton-Raphson IRLS
- **Ch. 5.1–5.3:** Backpropagation and error functions — rigorous derivation
- **Ch. 7:** SVM and its relationship to Gaussian processes
- **Ch. 9:** EM algorithm for GMMs — background for HMMs
- **Ch. 13:** HMMs — directly relevant to sequence recognition

---

## Key Equations (Chapter 3 — Linear Regression)

**Posterior over weights (Bayesian linear regression):**
$$p(w | t, X) = \mathcal{N}(w; m_N, S_N)$$
$$S_N^{-1} = S_0^{-1} + \beta X^\top X, \quad m_N = S_N(S_0^{-1}m_0 + \beta X^\top t)$$

**Predictive distribution:**
$$p(t^* | x^*, X, t) = \mathcal{N}(t^*; m_N^\top \phi(x^*), \sigma^2_N(x^*))$$
where variance $\sigma^2_N(x^*)$ includes both model uncertainty and noise — automatically quantifies prediction uncertainty.

---

## Why PRML is Irreplaceable

- **Probabilistic unification:** Every algorithm is a special case of Bayesian inference
- **Chapter 10** (Variational Inference) is the theoretical foundation for VAEs → [[Papers/Kingma2013_VAE|VAE]]
- **Chapter 13** (Sequential Data) covers HMMs exhaustively — relevant for sequence classification
- The book rewards re-reading at every stage of ML development

---

## Connections

- Bayesian regression → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- Logistic regression → [[Concepts/Logistic_Regression|Logistic Regression]]
- Neural networks → [[Concepts/Neural_Networks_MLP|Neural Networks & MLP]]
- SVM → [[Concepts/Kernel_Methods_SVM|Kernel Methods & SVM]]
- EM algorithm → [[Concepts/Clustering_EM|Clustering & EM Algorithm]]
- HMMs → [[Papers/Fink2014_MarkovModels|Fink 2014 (Markov Models)]]
- Variational inference → [[Foundations/Bayesian_Inference|Bayesian Inference]] (ELBO)
- Companion book → [[Papers/Hastie2009_ESL|Elements of Statistical Learning]] (more frequentist focus)
