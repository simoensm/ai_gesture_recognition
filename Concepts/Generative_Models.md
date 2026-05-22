---
title: "Generative Models: VAE, GAN, Diffusion"
tags: [concept, generative-models, vae, gan, diffusion, latent-space, deep-learning]
created: 2026-05-22
---

# Generative Models: VAE, GAN, Diffusion

> [!abstract] Learning the data distribution
> Discriminative models learn $P(y \mid x)$. Generative models learn $P(x)$ — the distribution of the data itself. This enables sampling, synthesis, anomaly detection, and compression.

---

## 1. Variational Autoencoder (VAE)

**Key paper:** [[Papers/Kingma2013_VAE|Kingma & Welling (2013)]]

**Model:** $z \sim P(z) = \mathcal{N}(0, I)$, $x \sim P_\theta(x \mid z)$

**Problem:** $P(x) = \int P_\theta(x \mid z) P(z)\, dz$ is intractable.

**Solution:** Variational inference. Introduce encoder $Q_\phi(z \mid x) \approx P(z \mid x)$.

**ELBO (Evidence Lower BOund):**
$$\log P_\theta(x) \geq \mathbb{E}_{Q_\phi(z \mid x)}[\log P_\theta(x \mid z)] - \text{KL}(Q_\phi(z \mid x) \| P(z))$$

- First term: **reconstruction loss** (how well the decoder recovers $x$)
- Second term: **regularisation** (pushes posterior toward the prior $\mathcal{N}(0,I)$)

**Reparameterisation trick:** $z = \mu_\phi(x) + \sigma_\phi(x) \cdot \varepsilon$, $\varepsilon \sim \mathcal{N}(0,I)$ — makes sampling differentiable.

**Properties:** Smooth, structured latent space; can interpolate between samples; output is often blurry.

---

## 2. Generative Adversarial Network (GAN)

**Key paper:** [[Papers/Goodfellow2014_GAN|Goodfellow et al. (2014)]]

**Two networks in adversarial training:**
- **Generator** $G(z)$: maps noise $z \sim P(z)$ to fake samples
- **Discriminator** $D(x)$: distinguishes real from fake

**Minimax objective:**
$$\min_G \max_D\, \mathbb{E}_{x \sim P_{\text{data}}}[\log D(x)] + \mathbb{E}_{z \sim P(z)}[\log(1 - D(G(z)))]$$

**At equilibrium:** $G$ generates samples from $P_{\text{data}}$; $D$ outputs 0.5 everywhere (can't distinguish).

**Information theory connection:** The original GAN objective minimises Jensen-Shannon divergence (JSD) between real and generated distributions → [[Foundations/Information_Theory|JSD]]

**Variants:**
- **WGAN:** Uses Wasserstein distance → more stable training
- **DCGAN:** Deep convolutional GAN for images
- **StyleGAN:** State-of-the-art face synthesis
- **cGAN:** Conditional generation with class label input

**Properties:** High-fidelity samples; mode collapse (generator ignores parts of distribution); training instability.

---

## 3. Diffusion Models (Score-based)

**Key papers:** Ho et al. (2020) DDPM; Song et al. (2020) Score Matching

**Two processes:**
- **Forward (diffusion):** Gradually add Gaussian noise to data over $T$ steps
  $$q(x_t \mid x_{t-1}) = \mathcal{N}(x_t;\, \sqrt{1-\beta_t}\, x_{t-1},\, \beta_t I)$$
- **Reverse (denoising):** Learn to remove noise step by step
  $$P_\theta(x_{t-1} \mid x_t) = \mathcal{N}(x_{t-1};\, \mu_\theta(x_t, t),\, \Sigma_\theta(x_t, t))$$

**Training:** Predict the noise $\varepsilon$ added at step $t$: $\mathcal{L} = \|\varepsilon - \varepsilon_\theta(x_t, t)\|^2$

**Properties:**
- **Highest quality** samples (Stable Diffusion, DALL-E 2, Midjourney all use diffusion)
- Slow inference (requires many denoising steps)
- Principled probabilistic framework

---

## 4. Comparison

| | VAE | GAN | Diffusion |
|---|---|---|---|
| **Sample quality** | Medium (blurry) | High | Highest |
| **Sample diversity** | High | Medium (mode collapse) | High |
| **Training stability** | Stable | Unstable | Stable |
| **Latent space** | Structured | Unstructured | Implicit |
| **Inference speed** | Fast | Fast | Slow |
| **Likelihood** | Tractable (ELBO) | Intractable | Tractable |

---

## Connections

- VAE uses → [[Foundations/Bayesian_Inference|ELBO, variational inference]], [[Foundations/Information_Theory|KL divergence]]
- GAN loss = JSD → [[Foundations/Information_Theory|Information Theory]]
- Diffusion models build on → [[Foundations/Probability_Theory|Gaussian distributions]]
- Both VAE and GAN use → [[Backpropagation]], [[Foundations/Optimisation_Theory|Adam]]
