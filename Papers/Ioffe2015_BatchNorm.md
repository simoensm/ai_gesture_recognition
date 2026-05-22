---
title: "Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift"
authors: ["Ioffe, Sergey", "Szegedy, Christian"]
year: 2015
citekey: "Ioffe2015_BatchNorm"
doi: "10.48550/arXiv.1502.03167"
venue: "ICML 2015, pp. 448–456"
type: conference
tags: [literature, batch-normalisation, deep-learning, training, internal-covariate-shift, regularisation, optimisation, bilstm]
status: read
relevance: high
created: 2026-05-22
---

# Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift

> [!info] Metadata
> **Authors:** Sergey Ioffe, Christian Szegedy (Google)
> **Year:** 2015
> **Venue:** ICML 2015, pp. 448–456
> **arXiv:** [1502.03167](https://arxiv.org/abs/1502.03167)

---

## Abstract

Introduces **Batch Normalisation (BN)** — a technique that normalises layer inputs within each mini-batch during training, dramatically reducing sensitivity to weight initialisation, allowing much higher learning rates, and acting as a regulariser. Reduced training time from weeks to hours on ImageNet and enabled training of very deep networks.

---

## Key Contributions

### Internal Covariate Shift

As training progresses, the distribution of each layer's inputs changes because the preceding layer's parameters change — called **internal covariate shift**. This forces each layer to continually adapt to a shifting input distribution, slowing training.

BN addresses this by normalising each layer's inputs to have zero mean and unit variance within each mini-batch.

### Batch Normalisation Transform

For a mini-batch $\mathcal{B} = \{x_1, \ldots, x_m\}$, for each feature dimension:

**Step 1 — Mini-batch statistics:**
$$\mu_\mathcal{B} = \frac{1}{m}\sum_{i=1}^m x_i, \qquad \sigma^2_\mathcal{B} = \frac{1}{m}\sum_{i=1}^m (x_i - \mu_\mathcal{B})^2$$

**Step 2 — Normalise:**
$$\hat{x}_i = \frac{x_i - \mu_\mathcal{B}}{\sqrt{\sigma^2_\mathcal{B} + \varepsilon}}$$

**Step 3 — Scale and shift (learnable parameters $\gamma, \beta$):**
$$y_i = \gamma \hat{x}_i + \beta$$

The learnable $\gamma$ and $\beta$ allow the network to recover the original distribution if that is optimal — BN never forces normalisation, it only removes covariate shift.

### At Inference
Use population statistics (exponential moving average of $\mu_\mathcal{B}$ and $\sigma^2_\mathcal{B}$ computed during training) instead of mini-batch statistics.

---

## Why Batch Norm Works

1. **Higher learning rates:** BN prevents activations from exploding or vanishing → can use $10\times$ larger learning rates
2. **Reduces sensitivity to initialisation:** The normalisation step undoes bad weight scales
3. **Implicit regularisation:** BN adds noise via batch statistics → acts like dropout (reduces need for strong dropout)
4. **Smooths the loss landscape:** Batch-normalised networks have a smoother, more well-conditioned loss surface → gradient descent converges faster

---

## Batch Norm for Sequences (BiLSTM)

Standard BN operates over the batch and spatial dimensions. For **sequential models** (RNNs, LSTMs):

- **Layer Normalization** (Ba et al., 2016) is preferred — normalises over the feature dimension *per sample*, not over the batch. No batch size dependence; works with variable-length sequences.
- **Batch Norm in 1D convolutions** over the temporal axis works well for TCNs (Temporal Convolutional Networks)

In the BiLSTM implementation: use **LayerNorm** between LSTM layers rather than BN:
```python
nn.LayerNorm(hidden_size * 2)  # after BiLSTM output
```

---

## Results

- ImageNet training: 14× fewer training steps to reach the same accuracy as a baseline model
- Allowed removal of dropout in some layers (BN provides sufficient regularisation)
- Enabled training of much deeper networks (predecessor to ResNet's success)

---

## Connections

- Used in → [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]] (as LayerNorm)
- Regularisation perspective → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Training acceleration → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- Dropout comparison → [[Papers/Srivastava2014_Dropout|Srivastava et al. (2014)]]
- Architecture context → [[Concepts/Neural_Networks_MLP|Neural Networks & MLP]]
- Loss landscape smoothing → [[Papers/He2016_ResNet|ResNet]] (same team, successive work)
