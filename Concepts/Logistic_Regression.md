---
title: "Logistic Regression"
tags: [concept, supervised-learning, logistic-regression, classification, sigmoid, softmax, cross-entropy, discriminative]
created: 2026-05-22
---

# Logistic Regression

> A discriminative linear classifier that outputs calibrated probabilities. The fundamental building block of neural networks — a single logistic unit is one neuron.

---

## 1. Binary Logistic Regression

### Model
$$p(y=1 | x; w) = \sigma(w^\top x + b) = \frac{1}{1 + e^{-(w^\top x + b)}}$$

The **sigmoid function** $\sigma(z) = 1/(1+e^{-z})$ maps the linear score to $[0,1]$:

```
σ(z)
 1 │         ___________
   │       /
0.5│ - - -/- - - - - - -
   │     /
 0 │____/
   └──────────────────── z
        0
```

**Decision boundary:** $\sigma(w^\top x + b) = 0.5$ iff $w^\top x + b = 0$ — a hyperplane.

### Training: Maximum Likelihood
The log-likelihood for $n$ binary observations:
$$\ell(w) = \sum_i \left[y_i \log\hat{p}_i + (1-y_i)\log(1-\hat{p}_i)\right]$$

Minimising **negative log-likelihood = binary cross-entropy loss:**
$$\mathcal{L}(w) = -\frac{1}{n}\sum_i \left[y_i \log \sigma(z_i) + (1-y_i)\log(1-\sigma(z_i))\right]$$

### Gradient
$$\nabla_w \mathcal{L} = \frac{1}{n}\sum_i (\hat{p}_i - y_i) x_i = \frac{1}{n}X^\top(\hat{p} - y)$$

No closed form — solved iteratively (gradient descent, Newton's method, IRLS).

### Properties of the sigmoid gradient
$$\sigma'(z) = \sigma(z)(1 - \sigma(z))$$
— vanishes for large $|z|$ — this is the **vanishing gradient** problem when stacking sigmoids in deep networks.

---

## 2. Multiclass Logistic Regression (Softmax)

For $K$ classes with weight vectors $w_1, \ldots, w_K \in \mathbb{R}^d$:

$$p(y=k | x; W) = \text{softmax}(Wx)_k = \frac{e^{w_k^\top x}}{\sum_{j=1}^K e^{w_j^\top x}}$$

The softmax is a smooth, differentiable generalisation of argmax that outputs a valid probability distribution over $K$ classes.

### Categorical cross-entropy loss
$$\mathcal{L}(W) = -\frac{1}{n}\sum_i\sum_k y_{ik}\log\hat{p}_{ik}$$

where $y_{ik} = \mathbb{1}[y_i = k]$ (one-hot encoding).

### Numerical stability
Computing $\text{softmax}(z)$ directly can overflow. Use:
$$\text{softmax}(z)_k = \frac{e^{z_k - z^*}}{\sum_j e^{z_j - z^*}}, \quad z^* = \max_j z_j$$

---

## 3. Connection to Linear Models and Neural Networks

**Logistic regression as a 1-layer neural network:**
```
x ──→ [w⊤x + b] ──→ σ(·) ──→ ŷ ∈ (0,1)
```

An MLP is logistic regression with hidden layers between the input and the output sigmoid/softmax. The output layer of *every* classification neural network is logistic/softmax regression.

**Decision boundary:** Linear (hyperplane). To get non-linear boundaries:
- Add features manually (polynomial features)
- Use kernel trick → [[Concepts/Kernel_Methods_SVM|SVM with kernel]]
- Stack hidden layers → [[Concepts/Neural_Networks_MLP|MLP]]

---

## 4. Regularisation

**L2-regularised (Ridge logistic):**
$$\mathcal{L}_\lambda(w) = \mathcal{L}(w) + \frac{\lambda}{2}\|w\|_2^2$$
→ MAP with Gaussian prior on $w$.

**L1-regularised (Sparse logistic):**
$$\mathcal{L}_\lambda(w) = \mathcal{L}(w) + \lambda\|w\|_1$$
→ Sparse weights; useful for feature selection in high-dimensional settings.

---

## 5. Generative vs. Discriminative Models

Logistic regression is **discriminative** — it directly models $p(y|x)$ without modelling $p(x|y)$.

| Aspect | Discriminative (Logistic Reg.) | Generative (Naive Bayes) |
|---|---|---|
| **Models** | $p(y \| x)$ directly | $p(x \| y)$ then Bayes' theorem |
| **Optimisation** | Conditional likelihood | Joint likelihood |
| **Asymptotic accuracy** | Better with more data | May be better with less data |
| **Missing features** | Problematic | Handles gracefully |
| **Robustness** | More robust to model mismatch | Assumes feature distribution |

Ng & Jordan (2002) showed that logistic regression = Naive Bayes in the limit of infinite data when the Naive Bayes assumptions hold.

---

## 6. Practical Notes

- **Convergence:** Use `lbfgs` solver for small-medium datasets; SGD for large
- **Class imbalance:** Use `class_weight='balanced'` or adjust decision threshold
- **Calibration:** Logistic regression is generally well-calibrated (unlike SVMs which require Platt scaling)
- **Interpretability:** Weights $w_j$ have a direct interpretation — $e^{w_j}$ is the odds ratio for feature $j$

---

## Connections

- Extends → [[Concepts/Linear_Regression|Linear Regression]]
- Foundation for → [[Concepts/Neural_Networks_MLP|Neural Networks / MLP]] (output layer)
- Discriminative counterpart → [[Concepts/Probabilistic_Classifiers|Naive Bayes (Generative)]]
- Loss function theory → [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]]
- MLE derivation → [[Foundations/Maximum_Likelihood_Estimation|MLE & MAP]]
- Regularisation → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Reference → [[Papers/Bishop2006_PRML|PRML Ch. 4]], [[Papers/Hastie2009_ESL|ESL Ch. 4]], [[Papers/Murphy2012_MLaPP|Murphy Ch. 8]]
