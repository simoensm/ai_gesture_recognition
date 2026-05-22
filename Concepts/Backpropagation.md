---
title: "Backpropagation & Automatic Differentiation"
tags: [concept, backpropagation, automatic-differentiation, gradients, chain-rule, neural-networks]
created: 2026-05-22
---

# Backpropagation & Automatic Differentiation

> [!abstract] The algorithm that makes deep learning possible
> Backpropagation efficiently computes the gradient of the loss w.r.t. all parameters using the chain rule. Without it, training neural networks with millions of parameters would be computationally infeasible.

**Key paper:** Rumelhart, Hinton & Williams (1986) — [[Papers/Rumelhart1986_Backprop|Rumelhart1986]]

---

## 1. The Chain Rule

For composite functions $L = f(g(h(x)))$:
$$\frac{dL}{dx} = \frac{dL}{df} \cdot \frac{df}{dg} \cdot \frac{dg}{dh} \cdot \frac{dh}{dx}$$

For neural networks, the "chain" is the sequence of layers.

---

## 2. Forward & Backward Passes

Consider a simple network: $\hat{y} = W_2 \sigma(W_1 x + b_1) + b_2$

**Forward pass:** compute activations layer by layer, store intermediate values (the computation graph).

**Backward pass (backprop):** propagate gradient from loss back through each layer using the chain rule.

For layer $l$ with input $z^{(l-1)}$, weights $W^{(l)}$, activation $a^{(l)} = \sigma(W^{(l)} z^{(l-1)})$:

$$\delta^{(l)} = \frac{\partial L}{\partial z^{(l)}} = (W^{(l+1)})^\top \delta^{(l+1)} \odot \sigma'(z^{(l)})$$

$$\frac{\partial L}{\partial W^{(l)}} = \delta^{(l)} (z^{(l-1)})^\top$$

---

## 3. Vanishing & Exploding Gradients

**Vanishing gradients:** When $\|\partial L / \partial \theta^{(l)}\| \to 0$ for early layers.
- Cause: repeated multiplication by values $< 1$ (sigmoid saturates → $\sigma' \approx 0$)
- Fix: ReLU activations, batch normalisation, skip connections (ResNet), LSTM gates

**Exploding gradients:** When $\|\partial L / \partial \theta^{(l)}\| \to \infty$.
- Cause: repeated multiplication by large values
- Fix: gradient clipping

→ LSTM solves vanishing gradients for sequences: [[Papers/Hochreiter1997|Hochreiter1997]]

---

## 4. Automatic Differentiation (Autograd)

Modern frameworks (PyTorch, JAX) implement **automatic differentiation** — not symbolic differentiation, not numerical differentiation. It traces operations at runtime and builds a computation graph, then backpropagates through it exactly.

**Forward mode:** efficient for few inputs, many outputs  
**Reverse mode (used in DL):** efficient for many inputs, few outputs (single scalar loss)

PyTorch: `loss.backward()` triggers reverse-mode autodiff through the computation graph.

---

## 5. Activation Functions & Their Gradients

| Activation | Formula | Gradient | Issue |
|---|---|---|---|
| **Sigmoid** | $1/(1+e^{-x})$ | $\sigma(1-\sigma)$ | Vanishing for $|x| \gg 0$ |
| **Tanh** | $(e^x-e^{-x})/(e^x+e^{-x})$ | $1 - \tanh^2$ | Vanishing for $|x| \gg 0$ |
| **ReLU** | $\max(0,x)$ | $1_{x>0}$ | Dying ReLU (dead neurons) |
| **Leaky ReLU** | $\max(\alpha x, x)$ | $1$ or $\alpha$ | No dying neurons |
| **GELU** | $x\Phi(x)$ | — | Used in transformers (BERT) |
| **Softmax** | $e^{x_i}/\sum_j e^{x_j}$ | Jacobian | Final classification layer |

---

## Connections

- Chain rule → [[Foundations/Linear_Algebra_for_ML|Jacobians]]
- Vanishing gradients → solved by LSTM → [[Papers/Hochreiter1997|Hochreiter1997]]
- Skip connections → [[Papers/He2016_ResNet|ResNet]]
- Used in → [[Concepts/Neural_Networks_MLP|Neural Networks]], [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]]
- Optimiser receives gradients → [[Foundations/Optimisation_Theory|Optimisation Theory]]
