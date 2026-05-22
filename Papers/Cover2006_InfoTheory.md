---
title: "Elements of Information Theory"
authors: ["Cover, Thomas M.", "Thomas, Joy A."]
year: 2006
citekey: "Cover2006_InfoTheory"
doi: "10.1002/047174882X"
venue: "Wiley-Interscience (2nd edition)"
type: book
tags: [literature, information-theory, entropy, mutual-information, kl-divergence, channel-capacity, data-compression, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Elements of Information Theory

> [!info] Metadata
> **Authors:** Thomas M. Cover, Joy A. Thomas
> **Year:** 2006 (2nd edition)
> **Publisher:** Wiley-Interscience

---

## Abstract

The standard reference textbook for information theory. Covers Shannon entropy, mutual information, channel capacity, data compression (Huffman, Lempel-Ziv), rate-distortion theory, and connections to statistics, probability, and machine learning. Essential background for understanding KL divergence in VAEs, cross-entropy loss, and mutual information regularisation.

---

## Key Concepts

### Shannon Entropy
$$H(X) = -\sum_x p(x)\log p(x) = \mathbb{E}[-\log p(X)]$$
Measures average uncertainty (bits for $\log_2$, nats for $\ln$).

### Joint and Conditional Entropy
$$H(X,Y) = H(X) + H(Y|X)$$
$$H(Y|X) = \mathbb{E}_{p(x,y)}[-\log p(Y|X)]$$

### KL Divergence (Relative Entropy)
$$\text{KL}(P \| Q) = \sum_x p(x)\log\frac{p(x)}{q(x)} \geq 0$$
- Not a distance (asymmetric)
- $\text{KL}(P\|Q) = 0 \iff P = Q$
- Appears in ELBO derivation for VAEs

### Mutual Information
$$I(X;Y) = \text{KL}(p(x,y) \| p(x)p(y)) = H(X) - H(X|Y)$$
Measures statistical dependence — zero iff $X \perp Y$.

### Cross-Entropy
$$H(p,q) = -\sum_x p(x)\log q(x) = H(p) + \text{KL}(p\|q)$$
Minimising cross-entropy loss = minimising KL divergence from data to model.

### Channel Capacity
$$C = \max_{p(x)} I(X;Y)$$
Maximum rate at which information can be transmitted reliably through a noisy channel (Shannon 1948).

### Data Processing Inequality
For Markov chain $X \to Y \to Z$: $I(X;Y) \geq I(X;Z)$
No processing can increase mutual information — features cannot contain more information than the raw input.

## Connections to ML

- Cross-entropy loss ← [[Foundations/Information_Theory|Information Theory]]
- ELBO = reconstruction − KL ← [[Papers/Kingma2013_VAE|VAE]]
- GAN JSD connection ← [[Papers/Goodfellow2014_GAN|GAN]]
- Minimum description length ← [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]]
- Feature selection via mutual information ← [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]

## Connections

- Full theory → [[Foundations/Information_Theory|Information Theory]]
- Bayesian perspective → [[Foundations/Bayesian_Inference|Bayesian Inference]]
- Used in → [[Papers/Kingma2013_VAE|VAE]] (KL term), [[Papers/Goodfellow2014_GAN|GAN]] (JSD)
