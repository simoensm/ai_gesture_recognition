---
title: "A Simple Framework for Contrastive Learning of Visual Representations"
authors: ["Chen, Ting", "Kornblith, Simon", "Norouzi, Mohammad", "Hinton, Geoffrey E."]
year: 2020
citekey: "Chen2020_SimCLR"
doi: "10.48550/arXiv.2002.05709"
venue: "ICML 2020"
type: conference
tags: [literature, contrastive-learning, self-supervised, representation-learning, simclr, vision, augmentation, deep-learning]
status: read
relevance: high
created: 2026-05-22
---

# A Simple Framework for Contrastive Learning of Visual Representations (SimCLR)

> [!info] Metadata
> **Authors:** Ting Chen, Simon Kornblith, Mohammad Norouzi, Geoffrey E. Hinton (Google Brain)
> **Year:** 2020
> **Venue:** ICML 2020
> **arXiv:** [2002.05709](https://arxiv.org/abs/2002.05709)

---

## Abstract

SimCLR presents a clean, minimal framework for self-supervised visual representation learning via contrastive loss. By learning representations that are invariant to strong data augmentations, SimCLR achieves 76.5% top-1 accuracy on ImageNet with a linear probe — closing much of the gap with supervised learning without using any labels.

---

## Key Contributions

### Framework Components

1. **Data augmentation:** Two random views $\tilde{x}_i, \tilde{x}_j$ of each image $x$ via a stochastic augmentation $T \sim \mathcal{T}$
2. **Encoder** $f(\cdot)$ — any backbone (ResNet-50 used); extracts representation $h = f(\tilde{x})$
3. **Projection head** $g(\cdot)$ — 2-layer MLP maps $h \to z \in \mathbb{R}^{128}$; **discarded at test time**
4. **Contrastive loss** on $z$

### NT-Xent Loss (Normalised Temperature-scaled Cross-Entropy)

For a mini-batch of $N$ images (producing $2N$ augmented views), for positive pair $(i, j)$:
$$\ell(i,j) = -\log \frac{\exp(\text{sim}(z_i,z_j)/\tau)}{\sum_{k=1}^{2N}\mathbb{1}_{[k\neq i]}\exp(\text{sim}(z_i,z_k)/\tau)}$$

Total loss:
$$\mathcal{L} = \frac{1}{2N}\sum_{k=1}^N \left[\ell(2k-1,2k) + \ell(2k,2k-1)\right]$$

- $\text{sim}(u,v) = u^\top v/(\|u\|\|v\|)$ — cosine similarity
- $\tau$ — temperature hyperparameter (default 0.5); lower $\tau$ makes the loss focus more on hard negatives

### Ablation Study Results

Key findings from ablation experiments:
1. **Augmentation composition matters:** Random crop + colour distortion are the most critical; without colour jitter, representations learn colour histograms instead of semantics
2. **Projection head helps:** The non-linear projection head improves representation quality by 3% — removing it hurts significantly
3. **Larger batch size:** More negatives per batch → better performance (uses batch size 4096-8192)
4. **Training longer:** 800 epochs vs 100 epochs improves by 3–4%
5. **Wider/deeper encoders:** Scale consistently improves representations

---

## Results

| Method | Labels used | Top-1 Acc (linear probe) |
|---|---|---|
| Supervised ResNet-50 | 100% | 76.5% |
| **SimCLR (ResNet-50)** | 0% | 69.3% |
| **SimCLR (ResNet-50 4×)** | 0% | 76.5% |
| SimCLR semi-supervised (1% labels) | 1% | 73.9% |

SimCLR with a 4× wider ResNet matches supervised ResNet-50 using zero labels for pre-training.

---

## Critical Insight: Why Contrastive Learning Works

The contrastive loss forces the encoder to be **invariant** to augmentations that don't change the semantic content (crops, colour, blur) while being **sensitive** to semantic differences between images. The model cannot take shortcuts (colour, texture) when strong colour jitter is applied — it must learn shape/structure.

**Connection to mutual information:** The contrastive loss is a lower bound on the mutual information $I(z_i; z_j)$ between the two views.

---

## SimCLR v2

Chen et al. (2020b) scales to larger models (ResNet-152 + SK connections), adds semi-supervised fine-tuning, and introduces knowledge distillation from the large model to a small one. Achieves 80.0% top-1 with 1% labels.

---

## Connections

- Self-supervised framework → [[Concepts/Semi_Supervised_Self_Supervised|Semi/Self-Supervised Learning]]
- Contrastive to other SSL methods (BYOL, MoCo, DINO) → [[Concepts/Semi_Supervised_Self_Supervised|Semi/Self-Supervised Learning]]
- Backbone → [[Concepts/Convolutional_Neural_Networks|CNN]] / [[Papers/He2016_ResNet|ResNet]]
- Information-theoretic view → [[Foundations/Information_Theory|Information Theory]]
- Hinton co-author → [[Papers/Krizhevsky2012_AlexNet|AlexNet]] (Hinton group)
