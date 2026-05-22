---
title: "Dimensionality Reduction"
tags: [concept, dimensionality-reduction, pca, t-sne, umap, autoencoder, manifold]
created: 2026-05-22
---

# Dimensionality Reduction

> [!abstract] The curse of dimensionality and its cure
> High-dimensional data is sparse, distances become meaningless, and models overfit. Dimensionality reduction finds a lower-dimensional representation that preserves the structure you care about.

---

## 1. Why Reduce Dimensions?

- **Curse of dimensionality:** Volume of a unit ball $\to 0$ as $d \to \infty$; distances concentrate around the same value
- **Visualisation:** Project to 2D/3D to see structure
- **Preprocessing:** Remove noise, redundant features → better generalisation
- **Compression:** Store and transmit data more efficiently

---

## 2. Linear Methods

### PCA (Principal Component Analysis)

Find directions of maximum variance:
$$\max_{v: \|v\|=1} \text{Var}(Xv) = v^\top \Sigma v$$

Solution: eigenvectors of $\Sigma = \frac{1}{n}X^\top X$ (or SVD of $X$).

**Compression:** Project $x \in \mathbb{R}^d$ to $z = V_k^\top (x - \mu) \in \mathbb{R}^k$ where $V_k$ = top-$k$ eigenvectors.

**Reconstruction:** $\hat{x} = V_k z + \mu$ — linear autoencoder.

**Choosing $k$:** Scree plot (eigenvalue elbow) or % variance explained threshold (e.g., 95%).

→ Full derivation: [[Foundations/Linear_Algebra_for_ML|Linear Algebra for ML]]

### Kernel PCA

Apply PCA in a feature space $\phi(x)$ via the kernel trick:
$$k(x, x') = \phi(x)^\top \phi(x')$$

Captures non-linear structure without explicitly computing $\phi(x)$.

### ICA (Independent Component Analysis)

Find components that are statistically independent (stronger than uncorrelated).
Useful for signal source separation (e.g., EEG, audio).

---

## 3. Non-linear / Manifold Methods

### t-SNE (van der Maaten & Hinton, 2008)

1. Convert pairwise distances to probabilities (Gaussian kernel): $p_{ij} \propto \exp(-\|x_i - x_j\|^2 / 2\sigma^2)$
2. In 2D, define similar probabilities with Student-t kernel: $q_{ij} \propto (1 + \|y_i - y_j\|^2)^{-1}$
3. Minimise $\text{KL}(P \| Q)$ by gradient descent on 2D positions $\{y_i\}$

**Strengths:** Excellent local structure preservation; beautiful cluster visualisations  
**Weaknesses:** Non-deterministic; doesn't preserve global structure; slow for large $n$; perplexity is sensitive

**Use for:** Visualising embeddings (word vectors, image features, gesture encodings)

### UMAP (McInnes et al., 2018)

Based on Riemannian geometry and fuzzy topology. Faster than t-SNE and better at preserving global structure.

$$\text{UMAP objective} = \sum_{e \in E} w_h(e) \log \frac{w_h(e)}{w_l(e)} + (1-w_h(e))\log\frac{1-w_h(e)}{1-w_l(e)}$$

**Advantages over t-SNE:** Faster, scales to millions of points, preserves more global structure, deterministic, supports new data points

**Use for:** Same as t-SNE but larger datasets

---

## 4. Autoencoders

Learn a bottleneck representation:
```
Input x → [Encoder] → z (latent code) → [Decoder] → x̂ ≈ x
```

Minimise reconstruction loss: $\mathcal{L} = \|x - \hat{x}\|^2$

**Variants:**
- **Denoising autoencoder:** Input corrupted with noise, reconstruct clean version → learns robust features
- **Sparse autoencoder:** Penalise non-zero activations in $z$ → sparse representations
- **Variational autoencoder (VAE):** $z \sim \mathcal{N}(\mu, \sigma^2)$, Bayesian framework → [[Papers/Kingma2013_VAE|VAE]]

---

## 5. Comparison

| Method | Linear? | Global structure | Speed | New points? | Best for |
|---|---|---|---|---|---|
| PCA | Yes | ✓ | Fast | ✓ | Preprocessing, compression |
| Kernel PCA | No | Partial | Medium | With Nyström | Non-linear, moderate N |
| t-SNE | No | ✗ | Slow | ✗ | 2D/3D visualisation |
| UMAP | No | ✓ | Fast | ✓ | Visualisation + downstream tasks |
| Autoencoder | No | Partial | Varies | ✓ | Learnable features, deep pipeline |

---

## Connections

- PCA foundation → [[Foundations/Linear_Algebra_for_ML|SVD]]
- KL divergence in t-SNE → [[Foundations/Information_Theory|Information Theory]]
- Autoencoder → [[Papers/Kingma2013_VAE|VAE]]
- Gesture preprocessing: PCA used in $3 Recognizer → [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer]]
- Visualising gesture embeddings: t-SNE or UMAP on BiLSTM hidden states
