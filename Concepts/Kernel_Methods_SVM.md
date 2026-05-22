---
title: "Kernel Methods & Support Vector Machines"
tags: [concept, svm, kernel-methods, classification, margin, kernel-trick]
created: 2026-05-22
---

# Kernel Methods & Support Vector Machines (SVM)

> [!abstract] Elegant geometry
> SVMs find the maximum-margin hyperplane separating classes. Kernel methods lift data into infinite-dimensional feature spaces to make non-linearly-separable data linearly separable — without ever computing the transformation explicitly.

**Key papers:** [[Papers/Vapnik1995_SLT|Vapnik 1995 (SLT)]], [[Papers/Cortes1995_SVM|Cortes & Vapnik 1995 (SVM)]]

---

## 1. Linear SVM (Hard Margin)

Find the hyperplane $w^\top x + b = 0$ that **maximises the margin** between classes:

$$\max_{w,b} \frac{2}{\|w\|} \quad \text{s.t.} \quad y_i(w^\top x_i + b) \geq 1 \quad \forall i$$

Equivalently:
$$\min_{w,b} \frac{1}{2}\|w\|^2 \quad \text{s.t.} \quad y_i(w^\top x_i + b) \geq 1$$

The **support vectors** are the training points closest to the hyperplane (on the margin). The decision boundary is determined entirely by support vectors.

---

## 2. Soft Margin SVM (C-SVM)

Allows some misclassification with slack variables $\xi_i \geq 0$:
$$\min_{w,b,\xi} \frac{1}{2}\|w\|^2 + C\sum_{i}\xi_i \quad \text{s.t.} \quad y_i(w^\top x_i + b) \geq 1 - \xi_i$$

- $C \to \infty$: hard margin (no misclassification allowed)
- $C \to 0$: maximise margin, tolerate many errors

**Hinge loss:** $\ell_{\text{hinge}}(y, \hat{y}) = \max(0, 1 - y\hat{y})$ — the SVM loss function.

---

## 3. The Kernel Trick

Instead of explicitly computing $\phi(x)$ (potentially infinite-dimensional), define a kernel:
$$k(x, x') = \langle \phi(x), \phi(x') \rangle$$

The SVM dual only requires $k(x_i, x_j)$ — not $\phi$ itself.

**Common kernels:**

| Kernel | Formula | Feature space |
|---|---|---|
| **Linear** | $x^\top x'$ | Original space |
| **Polynomial** | $(x^\top x' + c)^d$ | Degree-$d$ polynomial features |
| **RBF / Gaussian** | $\exp(-\|x-x'\|^2 / 2\sigma^2)$ | Infinite-dimensional |
| **Laplacian** | $\exp(-\|x-x'\|_1 / \sigma)$ | Infinite-dimensional |
| **String kernel** | Edit distance based | Sequences (strings) |

**Mercer's theorem:** A symmetric function $k$ is a valid kernel iff the kernel matrix $K_{ij} = k(x_i, x_j)$ is positive semidefinite.

---

## 4. Connection to Regularisation

The SVM primal objective $\frac{1}{2}\|w\|^2 + C\sum\xi_i$ = **L2-regularised hinge loss**.

This is equivalent to Ridge regression but with hinge loss instead of squared loss.

---

## 5. SVMs vs. Neural Networks

| | SVM | Deep NN |
|---|---|---|
| **Theory** | Strong (margin theory, VC) | Weaker guarantees |
| **Small data** | Excellent | Needs more data |
| **Large data** | Slow ($\mathcal{O}(n^2)$–$\mathcal{O}(n^3)$ training) | Scales well (SGD) |
| **Feature engineering** | Requires good features or kernel | Learns features automatically |
| **Calibrated probabilities** | Requires Platt scaling | Softmax gives probabilities |

---

## Connections

- Theoretical justification → [[Foundations/Statistical_Learning_Theory|VC theory, margin theory]]
- Kernel on time series → sequence kernels → useful for gesture classification
- Bayesian interpretation → Gaussian Process (kernel is covariance function)
- Hinge loss vs. cross-entropy → [[Foundations/Maximum_Likelihood_Estimation|MLE / loss functions]]
