---
title: "Probability Theory"
tags: [foundations, probability, statistics, random-variables]
created: 2026-05-22
---

# Probability Theory

> [!abstract] Why it matters for ML
> Every ML model is a probabilistic model. Loss functions are negative log-likelihoods. Regularisation is a prior. Dropout is a Bayesian approximation. Understanding probability makes every algorithm transparent.

---

## 1. Sample Space & Events

A **probability space** $(\Omega, \mathcal{F}, P)$ consists of:
- $\Omega$: sample space (all possible outcomes)
- $\mathcal{F}$: $\sigma$-algebra (events we can assign probability to)
- $P: \mathcal{F} \to [0,1]$: probability measure satisfying Kolmogorov's axioms:
  1. $P(A) \geq 0$
  2. $P(\Omega) = 1$
  3. Countable additivity: $P(\bigcup_i A_i) = \sum_i P(A_i)$ for disjoint events

---

## 2. Random Variables & Distributions

A **random variable** $X: \Omega \to \mathbb{R}$ maps outcomes to real numbers.

**Discrete:** $P(X = x)$ — probability mass function (PMF)  
**Continuous:** $f_X(x)$ — probability density function (PDF), where $P(a \leq X \leq b) = \int_a^b f_X(x)\,dx$

### Key Distributions

| Distribution | PMF / PDF | Mean | Variance | ML use |
|---|---|---|---|---|
| Bernoulli$(p)$ | $p^x(1-p)^{1-x}$ | $p$ | $p(1-p)$ | Binary classification |
| Categorical$(p)$ | $\prod_k p_k^{[x=k]}$ | — | — | Softmax output |
| Gaussian$(\mu, \sigma^2)$ | $\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$ | $\mu$ | $\sigma^2$ | Noise model, prior |
| Multivariate Gaussian | $\frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}}e^{-\frac{1}{2}(x-\mu)^\top\Sigma^{-1}(x-\mu)}$ | $\mu$ | $\Sigma$ | High-dim features |
| Laplace$(\mu, b)$ | $\frac{1}{2b}e^{-\frac{|x-\mu|}{b}}$ | $\mu$ | $2b^2$ | Sparse (L1) prior |

---

## 3. Expectation, Variance, Covariance

$$\mathbb{E}[X] = \int x\, f_X(x)\, dx \qquad \text{(continuous)}$$

$$\text{Var}(X) = \mathbb{E}[(X - \mathbb{E}[X])^2] = \mathbb{E}[X^2] - (\mathbb{E}[X])^2$$

$$\text{Cov}(X, Y) = \mathbb{E}[(X - \mathbb{E}X)(Y - \mathbb{E}Y)]$$

**Linearity of expectation:** $\mathbb{E}[aX + bY] = a\mathbb{E}[X] + b\mathbb{E}[Y]$ (always, even if $X, Y$ dependent)

---

## 4. Conditional Probability & Bayes' Theorem

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}$$

**Bayes' theorem:**
$$P(A \mid B) = \frac{P(B \mid A)\, P(A)}{P(B)}$$

**Total probability:**
$$P(B) = \sum_i P(B \mid A_i)\, P(A_i)$$

→ This is the foundation of [[Bayesian_Inference]].

---

## 5. Independence & Conditional Independence

$X \perp Y$: $P(X, Y) = P(X)P(Y)$  
$X \perp Y \mid Z$: $P(X, Y \mid Z) = P(X \mid Z)P(Y \mid Z)$

**Markov assumption:** $X_t \perp X_{1:t-2} \mid X_{t-1}$ — the foundation of Markov chains and HMMs (→ [[Course - Sequence Modeling/Markov Chain|Markov Chain]])

---

## 6. Key Inequalities

| Inequality | Statement | ML use |
|---|---|---|
| **Jensen's** | $f(\mathbb{E}[X]) \leq \mathbb{E}[f(X)]$ for convex $f$ | ELBO derivation in VAE |
| **Markov's** | $P(X \geq a) \leq \mathbb{E}[X]/a$ for $X \geq 0$ | Generalisation bounds |
| **Chebyshev's** | $P(|X - \mu| \geq k\sigma) \leq 1/k^2$ | Concentration of measure |

---

## 7. Law of Large Numbers & Central Limit Theorem

**LLN:** $\bar{X}_n \xrightarrow{a.s.} \mu$ — sample mean converges to true mean (justifies empirical risk minimisation)

**CLT:** $\sqrt{n}(\bar{X}_n - \mu) \xrightarrow{d} \mathcal{N}(0, \sigma^2)$ — justifies Gaussian approximations in inference

---

## Connections

- Distributions → [[Bayesian_Inference]] (posterior)
- Expectations → [[Maximum_Likelihood_Estimation]] (log-likelihood)
- Gaussians → [[Linear_Algebra_for_ML]] (covariance matrix)
- Entropy of distributions → [[Information_Theory]]
- Markov property → [[Course - Sequence Modeling/Markov Chain|Markov Chain]]
