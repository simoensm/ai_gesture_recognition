---
title: "Unsupervised Learning"
tags: [concept, unsupervised-learning, clustering, density-estimation, representation-learning, dimensionality-reduction, generative-models]
created: 2026-05-22
---

# Unsupervised Learning

> Learning structure from unlabelled data $\{x_1, \ldots, x_n\}$ — no targets $y$ provided. The goal is to discover hidden patterns, compress information, or learn representations that transfer to downstream tasks.

---

## 1. Why Unsupervised Learning?

- **Labels are expensive:** Collecting $n$ labelled examples costs $n \times \text{annotation cost}$. Unlabelled data is often abundant and cheap.
- **Structure discovery:** Revealing clusters, manifolds, or latent factors in data without prior knowledge of what categories exist
- **Representation learning:** Learning a feature space that makes downstream supervised tasks easier — the foundation of transfer learning and self-supervised learning
- **Density modelling:** Understanding $p(x)$ enables anomaly detection, data generation, and compression

---

## 2. Taxonomy of Unsupervised Methods

```
Unsupervised Learning
├── Clustering
│   ├── k-Means
│   ├── Gaussian Mixture Models (GMM)
│   ├── Hierarchical clustering (Ward, complete, single linkage)
│   └── Density-based (DBSCAN, HDBSCAN)
│
├── Dimensionality Reduction
│   ├── Linear: PCA, ICA, Factor Analysis, LDA*
│   └── Non-linear: t-SNE, UMAP, Autoencoders
│
├── Density Estimation
│   ├── Parametric: MLE of Gaussians, GMMs
│   ├── Non-parametric: Kernel Density Estimation (KDE)
│   └── Deep: Normalising Flows, VAEs, Diffusion Models
│
└── Representation Learning
    ├── Autoencoders (AE, VAE, Sparse AE)
    ├── Self-supervised (SimCLR, BYOL, MAE)
    └── Word Embeddings (Word2Vec, GloVe)
```
*LDA is supervised DR; included for comparison.

---

## 3. Clustering

### k-Means
Partition $n$ points into $K$ clusters by minimising within-cluster variance:
$$\min_{\mu_1,\ldots,\mu_K} \sum_{k=1}^K \sum_{i \in C_k} \|x_i - \mu_k\|^2$$

**Lloyd's algorithm:**
1. Initialise $K$ centroids (k-means++ for better initialisation)
2. Assign each point to the nearest centroid
3. Recompute centroids as cluster means
4. Repeat until convergence

Converges to a local minimum — run multiple times with different initialisations.

→ See [[Concepts/Clustering_EM|Clustering & EM Algorithm]] for full treatment.

### DBSCAN (Density-Based Spatial Clustering)
Groups points in high-density regions; marks low-density points as outliers.
- **Core point:** Has $\geq \text{minPts}$ neighbours within radius $\varepsilon$
- **Border point:** Within $\varepsilon$ of a core point but not itself a core point
- **Noise/outlier:** Not within $\varepsilon$ of any core point
- Discovers **arbitrarily shaped** clusters; does not require specifying $K$
- Hyperparameters: $\varepsilon$ (neighbourhood radius), minPts (density threshold)

### Hierarchical Clustering
Builds a dendrogram of nested clusters:
- **Agglomerative (bottom-up):** Start with $n$ singleton clusters; repeatedly merge the closest pair
- **Divisive (top-down):** Start with one cluster; recursively split
- **Linkage criteria:** Single (min distance), complete (max distance), average, Ward (minimise variance)

No need to pre-specify $K$ — cut the dendrogram at any level.

---

## 4. Dimensionality Reduction

Detailed coverage → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]

### PCA (Principal Component Analysis)
Find the $k$ orthogonal directions of maximum variance:
$$\max_{v: \|v\|=1} \text{Var}[v^\top x] = v^\top \Sigma v$$

Solution: top $k$ eigenvectors of the data covariance $\Sigma = \frac{1}{n}X^\top X$ — equivalently, the top $k$ left singular vectors from SVD.

### t-SNE
Non-linear dimensionality reduction for visualisation (2D/3D). Converts pairwise distances to probabilities; minimises KL divergence between high- and low-dimensional probability distributions. Excellent for discovering cluster structure visually; not suitable as features for downstream tasks.

### Autoencoders
$$x \xrightarrow{\text{encoder}} z \in \mathbb{R}^k \xrightarrow{\text{decoder}} \hat{x} \approx x, \quad k \ll d$$

Minimise reconstruction loss $\|x - \hat{x}\|^2$. The bottleneck $z$ is the compressed representation. Variants:
- **Sparse AE:** Regularise $z$ to be sparse → learns over-complete representations
- **Denoising AE:** Input corrupted with noise; reconstruct clean version → more robust features
- **VAE:** Probabilistic encoder $q_\phi(z|x) = \mathcal{N}(\mu_\phi, \sigma_\phi^2)$ → [[Papers/Kingma2013_VAE|VAE]]

---

## 5. Density Estimation

### Kernel Density Estimation (KDE)
Non-parametric estimate of $p(x)$:
$$\hat{p}(x) = \frac{1}{nh}\sum_{i=1}^n K\left(\frac{x - x_i}{h}\right)$$
where $K$ is a kernel (e.g., Gaussian) and $h$ is the bandwidth. As $h \to 0$: overfits (spiky). As $h \to \infty$: over-smooths.

### Gaussian Mixture Models (GMM)
$$p(x) = \sum_{k=1}^K \pi_k \mathcal{N}(x; \mu_k, \Sigma_k)$$
Trained via EM algorithm. More interpretable than KDE; assumes data comes from $K$ Gaussians.

---

## 6. Representation Learning

### Why Learn Representations?
A good representation makes the structure of data explicit so that **simple** downstream models (often linear) work well. Poor representation → complex model needed.

**Bengio et al. (2013)** identified desirable properties of learned representations: distributed (features shared across concepts), sparse, hierarchical, invariant to nuisance factors, disentangled.

### Word Embeddings
- **Word2Vec (Mikolov 2013):** Predict centre word from context (CBOW) or context from centre word (skip-gram). Produces dense embeddings capturing semantic relationships ($\text{king} - \text{man} + \text{woman} \approx \text{queen}$)
- **GloVe:** Factorises the word co-occurrence matrix

### Autoencoders (see §4 above)

### Self-supervised Learning → see [[Concepts/Semi_Supervised_Self_Supervised|Semi/Self-Supervised Learning]]

---

## 7. Evaluating Unsupervised Learning

**Clustering:**
- **Inertia / Within-cluster sum of squares:** Lower = tighter clusters (biased toward more clusters)
- **Silhouette score:** $s(i) = (b(i)-a(i))/\max(a(i),b(i)) \in [-1,1]$ — how much closer point $i$ is to its own cluster vs. the next nearest
- **Elbow method:** Plot inertia vs. $K$; pick the "elbow"
- **External:** If ground truth available, use Adjusted Rand Index (ARI), Normalised Mutual Information (NMI)

**Density estimation:** Log-likelihood on held-out test data.

**Representation quality:** Performance of a linear probe trained on top of the frozen representation.

---

## Connections

- Clustering details → [[Concepts/Clustering_EM|Clustering & EM Algorithm]]
- Dimensionality reduction → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]
- Generative models (VAE, GAN) → [[Concepts/Generative_Models|Generative Models]]
- Self-supervised learning → [[Concepts/Semi_Supervised_Self_Supervised|Semi/Self-Supervised Learning]]
- Contrasted with → [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]]
- References → [[Papers/Bishop2006_PRML|PRML Ch. 9–12]], [[Papers/Murphy2012_MLaPP|Murphy Ch. 11–12]], [[Papers/Hastie2009_ESL|ESL Ch. 8, 14]]
