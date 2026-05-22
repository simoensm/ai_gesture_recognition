---
title: "Dropout: A Simple Way to Prevent Neural Networks from Overfitting"
authors: ["Srivastava, Nitish", "Hinton, Geoffrey E.", "Krizhevsky, Alex", "Sutskever, Ilya", "Salakhutdinov, Ruslan"]
year: 2014
citekey: "Srivastava2014_Dropout"
doi: "10.5555/2627435.2670313"
venue: "Journal of Machine Learning Research, 15(1), 1929–1958"
type: article
tags: [literature, dropout, regularisation, deep-learning, overfitting, neural-networks, bilstm, foundational]
status: read
relevance: high
created: 2026-05-22
---

# Dropout: A Simple Way to Prevent Neural Networks from Overfitting

> [!info] Metadata
> **Authors:** Nitish Srivastava, Geoffrey E. Hinton, Alex Krizhevsky, Ilya Sutskever, Ruslan Salakhutdinov (University of Toronto)
> **Year:** 2014
> **Venue:** Journal of Machine Learning Research, 15(1), 1929–1958
> **Free PDF:** [jmlr.org/papers/v15/srivastava14a.html](https://jmlr.org/papers/v15/srivastava14a.html)

---

## Abstract

Introduces **dropout** — a simple and powerful regularisation technique for neural networks. During training, each neuron is independently set to zero with probability $p$ (the dropout rate). This prevents neurons from co-adapting and forces the network to learn redundant representations. At test time, all neurons are active but their outputs are scaled by $(1-p)$ to maintain the same expected activation.

---

## Key Contributions

### The Dropout Mechanism

**Training:** At each forward pass, for each neuron $j$ in layer $l$, draw $r_j^{(l)} \sim \text{Bernoulli}(1-p)$:
$$\tilde{a}_j^{(l)} = r_j^{(l)} \cdot a_j^{(l)}$$

The thinned network is used for both forward and backward passes in that step. Different neurons are dropped each batch.

**Inference (test time):** Use all neurons, scale weights by $(1-p)$:
$$W^*_{test} = (1-p) \cdot W$$

This ensures the expected output at test time equals the training-time expectation.

**Inverted dropout** (modern PyTorch default): scale activations by $1/(1-p)$ during training instead, so no scaling is needed at test time:
```python
nn.Dropout(p=0.3)  # PyTorch uses inverted dropout by default
```

### Theoretical Interpretation

**Ensemble view:** With $n$ neurons, dropout samples from $2^n$ possible "thinned" sub-networks, each sharing weights. The test-time network is an approximation of **geometric mean** over these exponentially many models — efficient model averaging.

**Bayesian view:** Dropout approximates a Bayesian neural network where the posterior over weights is approximated by a Bernoulli distribution over binary masks (Gal & Ghahramani, 2016).

**Co-adaptation prevention:** Without dropout, neurons can co-adapt to fix errors made by other neurons. Dropout forces each neuron to be independently useful.

---

## Dropout Rates — Practical Guide

| Layer type | Typical dropout rate $p$ |
|---|---|
| Input layer | 0.1–0.2 (light) |
| Hidden FC layers | 0.3–0.5 |
| LSTM hidden state | 0.2–0.4 (between layers, not within) |
| Recurrent connections | 0.0–0.2 (Variational Dropout; standard dropout on recurrent weights hurts) |
| Convolutional layers | 0.0–0.2 (or use Spatial Dropout) |

**For BiLSTM:** Apply dropout **between LSTM layers** (on the output of each layer), not on the recurrent connections within a layer. Using PyTorch's `nn.LSTM(dropout=0.3)` does this automatically for stacked LSTMs.

---

## Variational Dropout for RNNs (Gal & Ghahramani, 2016)

Standard dropout applied at each time step of an LSTM introduces inconsistent masks — different neurons are dropped at each step, disrupting the temporal gradient flow. **Variational dropout** applies the same mask across all time steps:

```python
# PyTorch doesn't support this natively; use custom LSTM
# The same mask is sampled once per forward pass and applied at all t
```

In practice, `nn.LSTM(dropout=p)` (inter-layer dropout) is sufficient for most gesture recognition tasks.

---

## Results

- MNIST: 1.35% error (best at the time without data augmentation)
- CIFAR-10: 12.6% error
- Consistently improved across vision, speech, NLP tasks
- Reduced overfitting on small datasets — directly applicable to the gesture recognition project where only 90 training sequences are available per fold

---

## Relevance to the Project

The BiLSTM baseline is trained on **90 sequences** (9 users × 10 classes × 1 sample per class... wait, 9 users × 10 classes × 10 reps = 900 training sequences per LOUO fold). Even with 900 training sequences, the model has hundreds of thousands of parameters — dropout is essential:
- Ablation: compare dropout rates 0.0, 0.2, **0.3**, 0.5 → [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Ablation Study MOC]]
- Standard value: `p = 0.3` between BiLSTM layers

---

## Connections

- Used in → [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]] (dropout between layers)
- Regularisation family → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Batch norm comparison → [[Papers/Ioffe2015_BatchNorm|Ioffe & Szegedy (2015)]]
- L1/L2 alternatives → [[Concepts/Linear_Regression|Linear Regression]] (ridge/lasso)
- Ensemble interpretation → [[Concepts/Decision_Trees_Ensembles|Ensembles]]
- Authors include → [[Papers/Krizhevsky2012_AlexNet|Krizhevsky]] (AlexNet) and [[Papers/Goodfellow2016|Hinton]] (Deep Learning)
