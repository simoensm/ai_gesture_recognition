---
title: "Maximum Likelihood Estimation (MLE & MAP)"
tags: [foundations, mle, map, statistics, loss-functions, estimation]
created: 2026-05-22
---

# Maximum Likelihood Estimation (MLE & MAP)

> [!abstract] Core insight
> Every standard loss function in ML is a negative log-likelihood. MSE = Gaussian likelihood. Cross-entropy = Bernoulli/categorical likelihood. Understanding MLE makes all loss functions interpretable.

---

## 1. The MLE Principle

Given data $\mathcal{D} = \{x_1, \ldots, x_n\}$ drawn i.i.d. from $P(x \mid \theta)$, find $\theta$ that makes the data most probable:

$$\hat{\theta}_{\text{MLE}} = \arg\max_\theta P(\mathcal{D} \mid \theta) = \arg\max_\theta \prod_{i=1}^n P(x_i \mid \theta)$$

Taking logs (monotone transformation, same argmax):
$$\hat{\theta}_{\text{MLE}} = \arg\max_\theta \sum_{i=1}^n \log P(x_i \mid \theta)$$

Equivalently, **minimise the negative log-likelihood (NLL)**:
$$\mathcal{L}(\theta) = -\frac{1}{n}\sum_{i=1}^n \log P(x_i \mid \theta)$$

---

## 2. MLE → Standard Loss Functions

### Regression: MSE = Gaussian NLL

Assume $y_i = f_\theta(x_i) + \varepsilon_i$ where $\varepsilon_i \sim \mathcal{N}(0, \sigma^2)$:
$$\log P(y_i \mid x_i, \theta) = -\frac{(y_i - f_\theta(x_i))^2}{2\sigma^2} + \text{const}$$

Maximising the log-likelihood = **minimising MSE**.

### Binary Classification: Cross-entropy = Bernoulli NLL

Assume $y_i \in \{0,1\}$, $P(y_i = 1 \mid x_i) = \hat{p}_i$ (sigmoid output):
$$\log P(y_i \mid x_i) = y_i \log \hat{p}_i + (1 - y_i) \log(1 - \hat{p}_i)$$

Maximising this = **minimising binary cross-entropy**.

### Multi-class Classification: Categorical NLL = Softmax Cross-entropy

$$\log P(y_i \mid x_i) = \sum_k \mathbf{1}[y_i = k] \log \hat{p}_{ik}$$

Maximising = **minimising categorical cross-entropy**. This is the standard loss for all classification networks.

---

## 3. MAP = MLE + Regularisation

With a prior $P(\theta)$:
$$\hat{\theta}_{\text{MAP}} = \arg\max_\theta \left[\sum_i \log P(x_i \mid \theta) + \log P(\theta)\right]$$

| Prior | $\log P(\theta)$ | Regularisation effect |
|---|---|---|
| Gaussian $\mathcal{N}(0, \lambda^{-1}I)$ | $-\frac{\lambda}{2}\|\theta\|_2^2$ | **L2 / Ridge / Weight Decay** |
| Laplace | $-\lambda\|\theta\|_1$ | **L1 / Lasso / Sparsity** |
| Uniform | 0 | No regularisation (= MLE) |

→ See also: [[Concepts/Bias_Variance_Regularisation|Overfitting & Regularisation]]

---

## 4. Properties of MLE

| Property | Statement |
|---|---|
| **Consistency** | $\hat{\theta}_{\text{MLE}} \xrightarrow{p} \theta^*$ as $n \to \infty$ |
| **Asymptotic normality** | $\sqrt{n}(\hat{\theta}_{\text{MLE}} - \theta^*) \xrightarrow{d} \mathcal{N}(0, \mathcal{I}(\theta^*)^{-1})$ |
| **Efficiency** | Achieves Cramér-Rao lower bound asymptotically |
| **Invariance** | If $\hat{\theta}$ is MLE of $\theta$, then $g(\hat{\theta})$ is MLE of $g(\theta)$ |

**Fisher Information Matrix:** $\mathcal{I}(\theta) = -\mathbb{E}\left[\nabla^2_\theta \log P(x \mid \theta)\right]$ — measures how much data tells you about $\theta$.

---

## 5. Connection to KL Divergence

MLE = minimise KL divergence between empirical distribution and model:
$$\hat{\theta}_{\text{MLE}} = \arg\min_\theta \text{KL}\left(\hat{P}_{\text{data}} \| P_\theta\right)$$

→ See [[Information_Theory]] for KL divergence details.

---

## Connections

- Loss functions in every model derive from here
- Regularisation ↔ MAP ↔ [[Bayesian_Inference]]
- KL divergence interpretation → [[Information_Theory]]
- Gradient-based optimisation of the NLL → [[Optimisation_Theory]]
- Logistic regression, neural networks, HMMs — all use MLE/NLL
