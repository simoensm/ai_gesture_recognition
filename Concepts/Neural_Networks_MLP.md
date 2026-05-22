---
title: "Neural Networks & Multi-Layer Perceptrons"
tags: [concept, neural-networks, mlp, deep-learning, feedforward, universal-approximation]
created: 2026-05-22
---

# Neural Networks & Multi-Layer Perceptrons (MLP)

> [!abstract] The universal approximator
> A sufficiently wide single hidden layer can approximate any continuous function (Universal Approximation Theorem). Deep networks approximate complex functions more efficiently with fewer parameters.

---

## 1. Architecture

**Perceptron (single unit):**
$$y = f(w^\top x + b) = f\left(\sum_{i=1}^d w_i x_i + b\right)$$

**Multi-Layer Perceptron:**
$$h^{(1)} = \sigma(W^{(1)} x + b^{(1)})$$
$$h^{(2)} = \sigma(W^{(2)} h^{(1)} + b^{(2)})$$
$$\hat{y} = \text{softmax}(W^{(L)} h^{(L-1)} + b^{(L)})$$

- $L$ layers, each with weight matrix $W^{(l)} \in \mathbb{R}^{n_l \times n_{l-1}}$
- Non-linear activation $\sigma$ between layers (ReLU most common)
- Final layer: sigmoid (binary), softmax (multi-class), linear (regression)

---

## 2. Universal Approximation Theorem

A single hidden layer MLP with a bounded, non-constant activation and enough hidden units can approximate any continuous function on a compact domain to arbitrary precision.

**Practical implication:** Width can compensate for depth — but depth is far more parameter-efficient for structured problems. Deep networks learn hierarchical features.

---

## 3. Capacity vs. Depth

| Architecture | Parameters | What it learns |
|---|---|---|
| Shallow wide | Many, one layer | Any function, but inefficiently |
| Deep narrow | Fewer, many layers | Hierarchical composition |
| Deep + wide | Most | Best in practice |

**Depth gives:** compositional structure (features of features), exponentially more functions representable per parameter.

---

## 4. Training Pipeline

```
Data (X, y)
    │
    ▼
Forward pass:  ŷ = f_θ(X)
    │
    ▼
Loss:          L = CrossEntropy(ŷ, y)
    │
    ▼
Backward pass: ∇_θ L  via backprop
    │
    ▼
Update:        θ ← θ − η · ∇_θ L  (Adam/SGD)
    │
    └── repeat for N epochs
```

---

## 5. Key Design Choices

| Choice | Options | Guidance |
|---|---|---|
| Width | 64, 128, 256, 512 | Match data complexity |
| Depth | 2–10 | Deeper = better for complex tasks |
| Activation | ReLU (default), GELU (transformers), Tanh (RNNs) | ReLU unless specific reason |
| Normalisation | BatchNorm, LayerNorm | BatchNorm for CNNs, LayerNorm for transformers/RNNs |
| Output | Softmax, Sigmoid, Linear | Match task |
| Regularisation | Dropout, Weight decay | See [[Bias_Variance_Regularisation]] |

---

## 6. MLP vs. Specialised Architectures

| Architecture | Inductive bias | Best for |
|---|---|---|
| MLP | None | Tabular data, final layers |
| CNN | Translation invariance | Images, local patterns |
| RNN/LSTM | Sequential order | Time series, sequences |
| Transformer | Global attention | NLP, long sequences |

---

## Connections

- Activation functions & vanishing gradients → [[Backpropagation]]
- Training → [[Foundations/Optimisation_Theory|Optimisation Theory]]
- Regularisation → [[Bias_Variance_Regularisation]]
- Sequential data → [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]]
- Images → [[Concepts/Convolutional_Neural_Networks|CNN]]
- Key papers: [[Papers/Rosenblatt1958_Perceptron|Perceptron (1958)]], [[Papers/Rumelhart1986_Backprop|Backprop (1986)]]
