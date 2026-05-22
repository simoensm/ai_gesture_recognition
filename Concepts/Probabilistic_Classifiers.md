---
title: "Probabilistic Classifiers — Naive Bayes, LDA & QDA"
tags: [concept, supervised-learning, naive-bayes, lda, qda, generative-model, bayes-theorem, probabilistic]
created: 2026-05-22
---

# Probabilistic Classifiers — Naive Bayes, LDA & QDA

> Generative classifiers model the joint distribution $p(x, y)$ and classify via Bayes' theorem. They often work well with limited data and handle missing features gracefully.

---

## 1. The Generative Classification Paradigm

**Bayes optimal classifier:**
$$\hat{y} = \arg\max_k p(y=k | x) = \arg\max_k \frac{p(x | y=k) \; p(y=k)}{p(x)}$$

Since $p(x)$ is constant across classes:
$$\hat{y} = \arg\max_k \underbrace{p(x|y=k)}_{\text{class-conditional likelihood}} \cdot \underbrace{p(y=k)}_{\text{prior}}$$

**Three things to specify:**
1. **Prior** $p(y)$ — estimated from class frequencies
2. **Likelihood** $p(x|y)$ — model assumption distinguishes Naive Bayes, LDA, QDA
3. **Inference** — apply Bayes' theorem

---

## 2. Naive Bayes Classifier

### Conditional Independence Assumption
$$p(x | y=k) = \prod_{j=1}^d p(x_j | y=k)$$

Assumes all features are **conditionally independent given the class** — often violated in practice, but the classifier is surprisingly robust.

### Decision Rule
$$\hat{y} = \arg\max_k \left[\log p(y=k) + \sum_{j=1}^d \log p(x_j | y=k)\right]$$

### Likelihood Models

**Gaussian Naive Bayes:** $p(x_j | y=k) = \mathcal{N}(x_j; \mu_{kj}, \sigma^2_{kj})$
- Estimate $\mu_{kj}$, $\sigma^2_{kj}$ from training data per class per feature
- Works for continuous features

**Multinomial Naive Bayes:** $p(x | y=k) = \text{Mult}(x; \phi_k)$ where $\phi_{kj} = p(\text{word } j | \text{class } k)$
- Classic for text classification (bag-of-words)
- Laplace smoothing to avoid zero probabilities: $\hat\phi_{kj} = (n_{kj}+1)/(n_k + d)$

**Bernoulli Naive Bayes:** Binary features $x_j \in \{0,1\}$
- Used for binary document features (word present/absent)

### When Naive Bayes Works Well
- Small datasets (few parameters to estimate)
- Text classification (despite feature dependence, works well)
- Real-time classification (extremely fast at inference)
- High-dimensional sparse features

---

## 3. Linear Discriminant Analysis (LDA)

### Assumptions
$$p(x | y=k) = \mathcal{N}(x; \mu_k, \Sigma)$$

**Shared covariance matrix** $\Sigma$ across all classes (only means differ).

### Decision Boundary
Comparing class $k$ and $j$:
$$\log\frac{p(k|x)}{p(j|x)} = x^\top \Sigma^{-1}(\mu_k - \mu_j) - \frac{1}{2}(\mu_k + \mu_j)^\top\Sigma^{-1}(\mu_k - \mu_j) + \log\frac{\pi_k}{\pi_j}$$

This is **linear in $x$** — LDA produces linear decision boundaries (like logistic regression).

### Parameter Estimation
$$\hat\mu_k = \frac{1}{n_k}\sum_{i: y_i=k} x_i, \quad \hat\Sigma = \frac{1}{n-K}\sum_k\sum_{i:y_i=k}(x_i - \hat\mu_k)(x_i-\hat\mu_k)^\top$$

### LDA as Dimensionality Reduction
LDA also defines a projection that maximises **between-class scatter** relative to **within-class scatter** — the Fisher criterion:
$$w^* = \arg\max_w \frac{w^\top S_B w}{w^\top S_W w}$$
where $S_B$ = between-class scatter matrix, $S_W$ = within-class scatter. Projects to at most $K-1$ dimensions.

---

## 4. Quadratic Discriminant Analysis (QDA)

### Assumptions
$$p(x | y=k) = \mathcal{N}(x; \mu_k, \Sigma_k)$$

**Class-specific covariance matrix** — removes the shared covariance constraint.

### Decision Boundary
**Quadratic in $x$** — QDA can learn curved (elliptical) decision boundaries.

### Comparison: LDA vs. QDA

| | LDA | QDA |
|---|---|---|
| **Covariance** | Shared $\Sigma$ | Per-class $\Sigma_k$ |
| **Boundary** | Linear (hyperplane) | Quadratic |
| **Parameters** | $Kd + d(d+1)/2$ | $Kd + Kd(d+1)/2$ |
| **Bias** | Higher (stricter assumption) | Lower |
| **Variance** | Lower (fewer params) | Higher |
| **Use when** | Small $n$, $d$; classes share spread | Larger $n$; classes have different shapes |

---

## 5. Generative vs. Discriminative: Deeper Comparison

The classic comparison (Ng & Jordan, 2002):

- **Generative (Naive Bayes, LDA):** Converges to a good solution faster (needs less data), but may plateau at a lower accuracy if the assumed model is wrong
- **Discriminative (Logistic Regression, SVM):** Asymptotically better when model is misspecified, but needs more data to converge

```
Accuracy
    │                    ──────  Discriminative
    │             ──────
    │       ──────
    │ ──────  Generative
    └───────────────────────── Training set size
```

In practice, for small datasets: **try Naive Bayes first**. For large datasets: **logistic regression or neural networks**.

---

## 6. Connections

- Bayes' theorem foundation → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- Probability distributions → [[Foundations/Probability_Theory|Probability Theory]]
- Discriminative counterpart → [[Concepts/Logistic_Regression|Logistic Regression]]
- Supervised framework → [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]]
- LDA as dimensionality reduction → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]
- Reference → [[Papers/Bishop2006_PRML|PRML Ch. 4]], [[Papers/Murphy2012_MLaPP|Murphy Ch. 3–4]], [[Papers/Hastie2009_ESL|ESL Ch. 4]]
