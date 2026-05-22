---
title: "Bayesian Inference"
tags: [foundations, bayesian, probability, inference, prior, posterior]
created: 2026-05-22
---

# Bayesian Inference

> [!abstract] Core idea
> Bayesian inference is learning by updating beliefs. You start with a **prior** belief about parameters, observe data, and compute a **posterior** belief. Every Bayesian algorithm is a version of this update.

---

## 1. The Bayesian Framework

$$\underbrace{P(\theta \mid \mathcal{D})}_{\text{posterior}} = \frac{\underbrace{P(\mathcal{D} \mid \theta)}_{\text{likelihood}} \cdot \underbrace{P(\theta)}_{\text{prior}}}{\underbrace{P(\mathcal{D})}_{\text{marginal likelihood}}}$$

| Term | Meaning | ML interpretation |
|---|---|---|
| $P(\theta)$ | Prior | Regularisation / inductive bias |
| $P(\mathcal{D} \mid \theta)$ | Likelihood | How well parameters explain data |
| $P(\theta \mid \mathcal{D})$ | Posterior | Updated belief after seeing data |
| $P(\mathcal{D})$ | Evidence | Normalising constant; used in model selection |

The **marginal likelihood** $P(\mathcal{D}) = \int P(\mathcal{D} \mid \theta) P(\theta)\, d\theta$ is usually intractable — this is why approximation methods exist.

---

## 2. Point Estimates from the Posterior

**MAP estimate** (Maximum A Posteriori):
$$\hat{\theta}_{\text{MAP}} = \arg\max_\theta P(\theta \mid \mathcal{D}) = \arg\max_\theta \left[\log P(\mathcal{D} \mid \theta) + \log P(\theta)\right]$$

**MLE** (Maximum Likelihood Estimate) — special case with uniform prior:
$$\hat{\theta}_{\text{MLE}} = \arg\max_\theta \log P(\mathcal{D} \mid \theta)$$

→ See [[Maximum_Likelihood_Estimation]] for full derivation.

**Regularisation as MAP:**
- Gaussian prior $P(\theta) = \mathcal{N}(0, \lambda^{-1}I)$ → MAP = MLE + L2 regularisation (Ridge)
- Laplace prior $P(\theta) \propto e^{-\lambda\|\theta\|_1}$ → MAP = MLE + L1 regularisation (Lasso)

---

## 3. Conjugate Priors

A prior is **conjugate** to a likelihood if the posterior has the same distributional form.

| Likelihood | Conjugate Prior | Posterior |
|---|---|---|
| Bernoulli / Binomial | Beta | Beta |
| Multinomial | Dirichlet | Dirichlet |
| Gaussian (known $\sigma$) | Gaussian | Gaussian |
| Gaussian (unknown $\sigma$) | Normal-Inverse-Gamma | Normal-Inverse-Gamma |
| Poisson | Gamma | Gamma |

Conjugate priors allow **closed-form Bayesian updates** — no numerical integration needed.

---

## 4. Approximate Inference

When the posterior is intractable:

**Variational Inference (VI):**
Approximate $P(\theta \mid \mathcal{D})$ with a simpler distribution $q(\theta)$ by minimising $\text{KL}(q \| p)$.
$$\mathcal{L}(\phi) = \mathbb{E}_{q_\phi(\theta)}[\log P(\mathcal{D} \mid \theta)] - \text{KL}(q_\phi(\theta) \| P(\theta))$$
This is the **ELBO** (Evidence Lower BOund) — directly used in [[Papers/Kingma2013_VAE|VAE]].

**MCMC (Markov Chain Monte Carlo):**
Sample from the posterior by constructing a Markov chain with stationary distribution = posterior. Gold standard for exact Bayesian inference; computationally expensive.

---

## 5. Bayesian Predictive Distribution

Instead of a point estimate, integrate over all parameters:
$$P(y^* \mid x^*, \mathcal{D}) = \int P(y^* \mid x^*, \theta)\, P(\theta \mid \mathcal{D})\, d\theta$$

This gives **calibrated uncertainty** in predictions — important for safe AI.

---

## 6. Naive Bayes Classifier

Applies Bayes' theorem with conditional independence assumption:
$$P(y \mid x_1, \ldots, x_d) \propto P(y) \prod_{j=1}^d P(x_j \mid y)$$

Despite the "naive" independence assumption, it works remarkably well in practice (text classification, spam filtering).

---

## Connections

- Foundation of → [[Maximum_Likelihood_Estimation]]
- ELBO + reparameterisation → [[Papers/Kingma2013_VAE|Variational Autoencoder (VAE)]]
- Prior = regularisation → [[Concepts/Bias_Variance_Regularisation|Overfitting & Regularisation]]
- Variational inference framework → [[Information_Theory]] (KL divergence)
- Dirichlet-Multinomial → [[Course - Sequence Modeling/HMM - Parameter Estimation (Baum-Welch)|Baum-Welch (HMM)]]
