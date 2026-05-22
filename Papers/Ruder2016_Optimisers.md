---
title: "An Overview of Gradient Descent Optimization Algorithms"
authors: ["Ruder, Sebastian"]
year: 2016
citekey: "Ruder2016_Optimisers"
doi: "10.48550/arXiv.1609.04747"
venue: "arXiv preprint"
type: preprint
tags: [literature, optimisation, gradient-descent, sgd, adam, momentum, rmsprop, review, deep-learning]
status: read
relevance: high
created: 2026-05-22
---

# An Overview of Gradient Descent Optimization Algorithms

> [!info] Metadata
> **Authors:** Sebastian Ruder
> **Year:** 2016
> **arXiv:** [1609.04747](https://arxiv.org/abs/1609.04747)

---

## Abstract

A comprehensive survey of gradient descent variants and adaptive optimizers used in deep learning. Covers SGD, Momentum, Nesterov, AdaGrad, RMSProp, Adam, and AdaMax, along with parallelisation strategies (Hogwild!, Downpour SGD) and strategies for choosing and tuning the learning rate.

---

## Gradient Descent Variants

### Batch vs. Stochastic vs. Mini-batch

| Type | Update on | Variance | Speed |
|---|---|---|---|
| Batch GD | All $n$ samples | Low | Slow (no parallelism advantage for small $n$) |
| SGD | 1 sample | High | Fast iterations, noisy |
| Mini-batch SGD | $B$ samples (e.g. 32–256) | Medium | Standard practice |

### Core Algorithms

**Momentum:** $v_t = \gamma v_{t-1} + \eta g_t$; $\theta \leftarrow \theta - v_t$
- Dampens oscillations, accelerates along consistent gradient directions

**Nesterov Accelerated Gradient (NAG):** Look ahead before computing gradient:
$g_t = \nabla L(\theta - \gamma v_{t-1})$ — theoretical $\mathcal{O}(1/t^2)$ convergence for convex problems

**AdaGrad:** $\theta_t = \theta_{t-1} - \frac{\eta}{\sqrt{G_t + \varepsilon}} g_t$ where $G_t = \sum_{i=1}^t g_i^2$
- Problem: $G_t$ only grows → learning rate decays to zero

**RMSProp:** Exponential moving average: $E[g^2]_t = \rho E[g^2]_{t-1} + (1-\rho)g_t^2$; $\theta_t = \theta_{t-1} - \frac{\eta}{\sqrt{E[g^2]_t + \varepsilon}}g_t$
- Fixes AdaGrad's monotonic decay

**Adam:** See → [[Papers/Kingma2014_Adam|Kingma & Ba 2014]]

**AdaMax:** Replace $\ell_2$ norm with $\ell_\infty$: $u_t = \max(\beta_2 u_{t-1}, |g_t|)$

## Practical Recommendations

- **Default:** Adam ($\alpha=0.001$) — robust for most tasks
- **Large-batch training:** SGD with momentum + learning rate warmup
- **Transformers:** AdamW with cosine annealing
- **Learning rate scheduling:** Step decay, cosine annealing, warmup + decay, cyclical LR

## Challenges in Optimisation

- Saddle points (not local minima) are the main obstruction in deep networks
- Gradient clipping prevents exploding gradients in RNNs
- Batch normalisation smooths the loss landscape, reducing sensitivity to learning rate

## Connections

- Adam → [[Papers/Kingma2014_Adam|Kingma & Ba 2014]]
- Gradient clipping, exploding gradients → [[Concepts/Backpropagation|Backpropagation]]
- Theory → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- Used in LSTM training → [[Papers/Hochreiter1997|LSTM]]
