---
title: "Auto-Encoding Variational Bayes (VAE)"
authors: ["Kingma, Diederik P.", "Welling, Max"]
year: 2013
citekey: "Kingma2013_VAE"
doi: "10.48550/arXiv.1312.6114"
venue: "ICLR 2014"
type: conference
tags: [literature, vae, variational-autoencoder, generative-models, bayesian, latent-space, deep-learning]
status: read
relevance: high
created: 2026-05-22
---

# Auto-Encoding Variational Bayes (VAE)

> [!info] Metadata
> **Authors:** Diederik P. Kingma, Max Welling
> **Year:** 2013
> **Venue:** ICLR 2014
> **arXiv:** [1312.6114](https://arxiv.org/abs/1312.6114)

---

## Abstract

Introduces the Variational Autoencoder — a generative model that combines deep neural networks with variational Bayesian inference. The encoder maps inputs to a latent distribution $Q_\phi(z \mid x) = \mathcal{N}(\mu_\phi(x), \text{diag}(\sigma_\phi^2(x)))$; the decoder samples from this and reconstructs the input. The **reparameterisation trick** makes the latent sampling differentiable.

---

## Key Contributions

- **ELBO objective:** Reconstruction loss + KL regularisation (analytical closed form for Gaussian prior)
$$\mathcal{L}(\phi, \theta) = \mathbb{E}_{Q_\phi(z|x)}[\log P_\theta(x|z)] - \text{KL}(Q_\phi(z|x) \| P(z))$$
- **Reparameterisation trick:** $z = \mu + \sigma \odot \varepsilon$, $\varepsilon \sim \mathcal{N}(0,I)$ — enables backprop through sampling
- Structured, continuous **latent space** → smooth interpolation between data points
- Foundation for modern generative models and representation learning

## Connection to Project

A VAE could be used to learn a compressed representation of gesture trajectories:
- Encode gesture → $z \in \mathbb{R}^k$ (latent code, typically $k \ll 3T$)
- Classify in latent space (smaller, cleaner features)
- Augment training data by sampling from latent space

## Connections

- Bayesian framework → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- KL divergence → [[Foundations/Information_Theory|Information Theory]]
- Compared to → [[Concepts/Generative_Models|GAN, Diffusion]]
- Reparameterisation → [[Concepts/Backpropagation|Automatic differentiation]]
- Same author as → [[Papers/Kingma2014_Adam|Adam optimiser]]
