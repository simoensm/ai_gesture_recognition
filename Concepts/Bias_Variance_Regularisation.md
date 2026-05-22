---
title: "Overfitting, Underfitting & Regularisation"
tags: [concept, bias-variance, regularisation, overfitting, dropout, l1-l2]
created: 2026-05-22
---

# Overfitting, Underfitting & Regularisation

> [!abstract] The central practical problem in ML
> A model that memorises training data perfectly but fails on new data is useless. Regularisation is the set of techniques that prevent this.

---

## 1. The Three Regimes

| Regime | Training error | Test error | Problem | Fix |
|---|---|---|---|---|
| **Underfitting** | High | High | Model too simple, can't capture patterns | More capacity, better features |
| **Just right** | Low | Low | Good generalisation | — |
| **Overfitting** | Very low | High | Model memorises noise | Regularise, more data, simpler model |

---

## 2. Sources of Test Error

$$\text{Test error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible noise}$$

- **Bias:** systematic error from wrong assumptions (underfitting)
- **Variance:** sensitivity to training data fluctuations (overfitting)
- **Noise:** inherent randomness in data — cannot be reduced

→ Full derivation in [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]]

---

## 3. L2 Regularisation (Ridge / Weight Decay)

$$\mathcal{L}_{\text{reg}} = \mathcal{L}_{\text{original}} + \frac{\lambda}{2}\|\theta\|_2^2$$

**Effect:** Penalises large weights → pushes all weights toward zero → smoother, less overfit model.

**Bayesian interpretation:** Gaussian prior $\theta \sim \mathcal{N}(0, \lambda^{-1}I)$ → MAP estimate.

In PyTorch: `optimizer = AdamW(model.parameters(), weight_decay=λ)`

---

## 4. L1 Regularisation (Lasso)

$$\mathcal{L}_{\text{reg}} = \mathcal{L}_{\text{original}} + \lambda\|\theta\|_1$$

**Effect:** Drives many weights exactly to zero → **sparse models** → implicit feature selection.

**Bayesian interpretation:** Laplace prior.

---

## 5. Dropout

At each training step, randomly zero out each neuron with probability $p$ (typically 0.2–0.5). → [[Papers/Srivastava2014_Dropout|Srivastava et al. (2014)]]

**Effect:**
- Prevents co-adaptation of neurons
- Equivalent to training an exponential ensemble of sub-networks
- At test time: scale activations by $(1-p)$ (or use inverted dropout during training)

**Bayesian interpretation (Gal & Ghahramani, 2016):** Dropout ≈ variational Bayesian inference in deep networks — uncertainty estimate for free.

Used in the BiLSTM: `dropout=0.3` between layers → [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]]

---

## 6. Early Stopping

Stop training when validation loss starts increasing. Effectively limits the model's capacity without changing the architecture.

Implements a form of L2 regularisation implicitly.

---

## 7. Data Augmentation

Generate additional training samples by applying label-preserving transformations:
- **Images:** flip, crop, rotate, color jitter (used in AlexNet, ResNet training)
- **Time series:** noise injection, time-warping, scaling

For gesture recognition: time-stretching (essentially what DTW alignment does) or adding noise.

---

## 8. Batch Normalisation

Normalise activations within each mini-batch: $\hat{x} = (x - \mu_B)/\sigma_B$, then scale/shift.

**Effects:**
- Reduces internal covariate shift
- Acts as mild regulariser (noisy $\mu_B, \sigma_B$ estimates)
- Allows higher learning rates
- Makes training more stable

---

## Connections

- Theory → [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]]
- L2 = Gaussian prior → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- Dropout in BiLSTM → [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]]
- Cross-validation to detect overfitting → [[Cross_Validation|Cross-Validation Protocol]]
