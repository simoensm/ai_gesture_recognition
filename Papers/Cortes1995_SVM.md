---
title: "Support-Vector Networks"
authors: ["Cortes, Corinna", "Vapnik, Vladimir N."]
year: 1995
citekey: "Cortes1995_SVM"
doi: "10.1023/A:1022627411411"
venue: "Machine Learning, 20(3), 273–297"
type: article
tags: [literature, svm, kernel, classification, margin, support-vectors, foundational]
status: read
relevance: high
created: 2026-05-22
---

# Support-Vector Networks

> [!info] Metadata
> **Authors:** Corinna Cortes, Vladimir N. Vapnik (AT&T Bell Labs)
> **Year:** 1995
> **Venue:** Machine Learning, 20(3), 273–297

---

## Abstract

Introduces the **Support Vector Machine** as a practical learning machine that constructs a decision surface with maximum margin — the hyperplane with the largest distance to the nearest training examples. The **kernel trick** extends SVMs to non-linearly separable data without explicit feature maps.

---

## Key Contributions

- **Hard-margin SVM:** Maximise $2/\|w\|$ (equivalently minimise $\frac{1}{2}\|w\|^2$) subject to correct classification constraints
- **Soft-margin SVM ($C$-SVM):** Introduce slack variables $\xi_i \geq 0$:
$$\min_{w,b,\xi} \frac{1}{2}\|w\|^2 + C\sum_i \xi_i \quad \text{s.t. } y_i(w^\top x_i + b) \geq 1 - \xi_i$$
  Parameter $C$ controls margin–misclassification trade-off
- **Kernel trick:** Replace $x_i^\top x_j$ with $K(x_i, x_j) = \phi(x_i)^\top \phi(x_j)$ — implicitly maps data to a high-dimensional feature space
- Common kernels: polynomial $K = (x^\top x' + c)^d$, RBF $K = \exp(-\gamma\|x-x'\|^2)$

## Why Kernel Trick Works

The SVM dual only involves dot products $x_i^\top x_j$. Replacing these with a kernel $K(x_i, x_j)$ is equivalent to applying a (potentially infinite-dimensional) feature map $\phi$ — without ever computing $\phi$ explicitly. **Mercer's theorem** guarantees $K$ must be a valid positive semi-definite kernel.

## Strengths and Limitations

**Strengths:**
- Strong theoretical guarantees (VC dimension, margin theory)
- Works well in high dimensions with limited data
- Non-parametric — decision boundary determined by support vectors only

**Limitations:**
- $\mathcal{O}(n^2)$–$\mathcal{O}(n^3)$ training complexity
- Kernel choice and hyperparameters ($C$, $\gamma$) are critical
- Does not naturally output probabilities (Platt scaling needed)

## Connections

- Theoretical foundation → [[Papers/Vapnik1995_SLT|Vapnik 1995 (Statistical Learning Theory)]]
- Kernel methods → [[Concepts/Kernel_Methods_SVM|Kernel Methods & SVM]]
- Alternative to → [[Concepts/Neural_Networks_MLP|Neural Networks]] for small-data regimes
- Generalisation bounds → [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]]
