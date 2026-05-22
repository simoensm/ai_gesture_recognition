---
title: "Generative Adversarial Nets"
authors: ["Goodfellow, Ian J.", "Pouget-Abadie, Jean", "Mirza, Mehdi", "Xu, Bing", "Warde-Farley, David", "Ozair, Sherjil", "Courville, Aaron", "Bengio, Yoshua"]
year: 2014
citekey: "Goodfellow2014_GAN"
doi: "10.48550/arXiv.1406.2661"
venue: "NeurIPS 2014"
type: conference
tags: [literature, gan, generative-models, adversarial-training, deep-learning, minimax, unsupervised]
status: read
relevance: high
created: 2026-05-22
---

# Generative Adversarial Nets

> [!info] Metadata
> **Authors:** Ian J. Goodfellow et al. (Montréal / MILA)
> **Year:** 2014
> **Venue:** NeurIPS 2014
> **arXiv:** [1406.2661](https://arxiv.org/abs/1406.2661)

---

## Abstract

Introduces the **Generative Adversarial Network (GAN)** framework — two neural networks (a Generator $G$ and a Discriminator $D$) trained simultaneously via an adversarial minimax game. The generator tries to fool the discriminator; the discriminator tries to distinguish real from generated samples. At equilibrium, $G$ produces samples indistinguishable from the data distribution.

---

## Key Contributions

- **Minimax objective:**
$$\min_G \max_D \; \mathbb{E}_{x \sim p_\text{data}}[\log D(x)] + \mathbb{E}_{z \sim p_z}[\log(1 - D(G(z)))]$$
- At optimality, $D^*(x) = \frac{p_\text{data}(x)}{p_\text{data}(x) + p_g(x)}$ and $p_g = p_\text{data}$
- Global optimum is achieved iff $p_g = p_\text{data}$, and the optimal $D$ value is $-\log 4$
- The generator loss connects to **Jensen-Shannon divergence**: $JSD(p_\text{data} \| p_g)$
- No need to explicitly model $p(x)$ — implicit generative model via sampling $z \sim \mathcal{N}(0,I)$

## Training Dynamics

- **Mode collapse** — a major failure mode: generator learns to produce a few modes that fool $D$
- **Training instability** — $G$ and $D$ must be balanced; if $D$ wins too early, gradients vanish for $G$
- Practical fix: train $D$ for $k$ steps per $G$ step
- Later work (WGAN, ProgressiveGAN, StyleGAN) addressed these instabilities

## Comparison with VAE

| | VAE | GAN |
|---|---|---|
| **Training** | ELBO maximisation | Minimax game |
| **Latent space** | Explicit, structured | Implicit (noise vector) |
| **Sample quality** | Blurry (MSE tendency) | Sharp (adversarial) |
| **Likelihood** | Tractable (approximate) | Intractable |
| **Mode coverage** | Better | Mode collapse risk |

## Connections

- Information-theoretic analysis → [[Foundations/Information_Theory|Information Theory]] (JSD)
- Compared to → [[Concepts/Generative_Models|VAE, Diffusion models]]
- Training uses → [[Concepts/Backpropagation|Backpropagation]]
- Optimiser typically → [[Papers/Kingma2014_Adam|Adam]]
- Later extensions: DCGAN, WGAN, CycleGAN, StyleGAN, BigGAN
