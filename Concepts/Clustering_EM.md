---
title: "Clustering, EM Algorithm & Gaussian Mixture Models"
tags: [concept, clustering, k-means, gmm, em-algorithm, unsupervised-learning]
created: 2026-05-22
---

# Clustering, EM Algorithm & Gaussian Mixture Models

> [!abstract] Learning without labels
> Clustering groups similar data points without supervision. The EM algorithm generalises k-means to soft probabilistic assignments and is the engine behind GMMs, HMMs (Baum-Welch), and many other latent variable models.

---

## 1. k-Means Clustering

**Objective:** Minimise total within-cluster variance:
$$\min_{\{c_k\}, \{S_k\}} \sum_{k=1}^K \sum_{x_i \in S_k} \|x_i - c_k\|^2$$

**Lloyd's algorithm (EM-like):**
1. **E step:** Assign each point to the nearest centroid: $z_i = \arg\min_k \|x_i - c_k\|^2$
2. **M step:** Recompute centroids: $c_k = \frac{1}{|S_k|}\sum_{i \in S_k} x_i$
3. Repeat until convergence.

**Properties:**
- Converges to a local minimum (not global)
- Sensitive to initialisation → use k-means++ for better starts
- Assumes spherical, equal-size clusters (Euclidean distance)

**Used in this project:** [[Concepts/Vector_Quantization|Vector Quantization]] for edit distance baseline — `GestureCodebook` is k-means.

---

## 2. Gaussian Mixture Models (GMM)

Soft version of k-means. Model data as a mixture of $K$ Gaussians:
$$P(x) = \sum_{k=1}^K \pi_k\, \mathcal{N}(x \mid \mu_k, \Sigma_k)$$

Parameters: mixing weights $\pi_k$, means $\mu_k$, covariances $\Sigma_k$.

**Why "soft":** Each point has a probability of belonging to each cluster (responsibility $r_{ik}$), not a hard assignment.

---

## 3. The EM Algorithm

The **Expectation-Maximisation (EM)** algorithm maximises the marginal log-likelihood when there are latent variables $z$:
$$\log P(\mathcal{D} \mid \theta) = \log \sum_z P(\mathcal{D}, z \mid \theta)$$

**Algorithm:**
- **E step:** Compute $Q(\theta, \theta_{\text{old}}) = \mathbb{E}_{z \mid \mathcal{D}, \theta_{\text{old}}}[\log P(\mathcal{D}, z \mid \theta)]$
- **M step:** $\theta_{\text{new}} = \arg\max_\theta Q(\theta, \theta_{\text{old}})$
- Repeat until convergence.

**Guarantee:** Each EM step increases (or keeps equal) the marginal log-likelihood. Converges to a local maximum.

### EM for GMM

**E step (responsibilities):**
$$r_{ik} = P(z_i = k \mid x_i, \theta) = \frac{\pi_k \mathcal{N}(x_i \mid \mu_k, \Sigma_k)}{\sum_{k'} \pi_{k'} \mathcal{N}(x_i \mid \mu_{k'}, \Sigma_{k'})}$$

**M step:**
$$\pi_k = \frac{1}{n}\sum_i r_{ik}, \quad \mu_k = \frac{\sum_i r_{ik} x_i}{\sum_i r_{ik}}, \quad \Sigma_k = \frac{\sum_i r_{ik}(x_i-\mu_k)(x_i-\mu_k)^\top}{\sum_i r_{ik}}$$

---

## 4. EM and HMMs

The Baum-Welch algorithm for HMM parameter estimation **is EM** applied to a hidden Markov model:
- **E step** = forward-backward algorithm (computes responsibilities for hidden states)
- **M step** = re-estimate transition/emission probabilities

→ [[Course - Sequence Modeling/HMM - Parameter Estimation (Baum-Welch)|Baum-Welch (HMM)]]

---

## 5. Choosing $K$

| Method | Criterion |
|---|---|
| **Elbow method** | Plot inertia vs. $K$, pick the "elbow" |
| **Silhouette score** | Measures how well-separated clusters are |
| **BIC/AIC for GMM** | Penalised likelihood criterion |
| **Gap statistic** | Compare to random baseline |

---

## Connections

- k-Means used as → [[Concepts/Vector_Quantization|Vector Quantization]] in edit distance
- EM generalises k-Means (soft assignments)
- EM = Baum-Welch → [[Course - Sequence Modeling/HMM - Parameter Estimation (Baum-Welch)|Baum-Welch]]
- EM related to → [[Foundations/Bayesian_Inference|Bayesian inference]] (ELBO)
- GMM relates to → [[Foundations/Probability_Theory|Gaussian distribution]]
