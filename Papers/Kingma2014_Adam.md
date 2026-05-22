---
title: "Adam: A Method for Stochastic Optimization"
authors: ["Kingma, Diederik P.", "Ba, Jimmy"]
year: 2014
citekey: "Kingma2014_Adam"
doi: "10.48550/arXiv.1412.6980"
venue: "ICLR 2015"
type: conference
tags: [literature, adam, optimisation, deep-learning, adaptive-learning-rates, momentum, rmsprop, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Adam: A Method for Stochastic Optimization

> [!info] Metadata
> **Authors:** Diederik P. Kingma, Jimmy Ba
> **Year:** 2014
> **Venue:** ICLR 2015
> **arXiv:** [1412.6980](https://arxiv.org/abs/1412.6980)

---

## Abstract

Proposes **Adam** (Adaptive Moment Estimation) — an optimizer that combines momentum (first moment) with adaptive per-parameter learning rates (second moment). Bias-corrected estimates prevent the common early-training collapse seen in RMSProp. Now the default optimizer for deep learning.

---

## Key Contributions

Given gradient $g_t$ at step $t$:

1. **First moment** (mean): $m_t = \beta_1 m_{t-1} + (1-\beta_1)g_t$
2. **Second moment** (uncentred variance): $v_t = \beta_2 v_{t-1} + (1-\beta_2)g_t^2$
3. **Bias correction:** $\hat{m}_t = m_t/(1-\beta_1^t)$, $\quad\hat{v}_t = v_t/(1-\beta_2^t)$
4. **Update:** $\theta_t = \theta_{t-1} - \alpha \cdot \hat{m}_t / (\sqrt{\hat{v}_t} + \varepsilon)$

**Default hyperparameters:** $\alpha=0.001$, $\beta_1=0.9$, $\beta_2=0.999$, $\varepsilon=10^{-8}$

- **Memory:** $\mathcal{O}(d)$ extra storage for $m$ and $v$ — no matrix quantities
- **Sparsity-friendly:** Adaptive rates benefit parameters with infrequent but large gradients (e.g., embeddings)

## Why Bias Correction Matters

At $t=1$, $m_1 = (1-\beta_1)g_1 \approx 0.1 g_1$ — massively downscaled. The bias correction $(1-\beta_1^t)$ restores the correct scale, preventing artificially small initial steps.

## Comparison

| Optimizer | Momentum | Adaptive LR | Bias correction |
|---|---|---|---|
| SGD | ✗ | ✗ | ✗ |
| Momentum SGD | ✓ | ✗ | ✗ |
| RMSProp | ✗ | ✓ | ✗ |
| **Adam** | ✓ | ✓ | ✓ |
| AdamW | ✓ | ✓ | ✓ + decoupled WD |

## Notes

- **AdamW** decouples weight decay from the adaptive step — preferred for transformers (Loshchilov & Hutter, 2019)
- AMSGrad variant provides convergence guarantees for non-convex settings

## Connections

- Theory → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- Used in → [[Papers/Kingma2013_VAE|VAE]], [[Papers/Vaswani2017_Transformer|Transformers]], [[Papers/Devlin2018_BERT|BERT]], and virtually all DL
- Same first author as → [[Papers/Kingma2013_VAE|VAE (Kingma & Welling, 2013)]]
- Gradient overview → [[Papers/Ruder2016_Optimisers|Ruder 2016 (Gradient Descent Overview)]]
