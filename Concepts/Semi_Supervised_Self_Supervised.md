---
title: "Semi-Supervised & Self-Supervised Learning"
tags: [concept, semi-supervised, self-supervised, contrastive-learning, pretraining, representation-learning, simclr, mae]
created: 2026-05-22
---

# Semi-Supervised & Self-Supervised Learning

> Paradigms that exploit abundant **unlabelled** data to improve learning when labels are scarce. The most important development in ML since 2018.

---

## 1. The Labelling Bottleneck

In many real-world problems:
- **Unlabelled data:** Cheap and abundant (web images, text corpora, sensor readings)
- **Labelled data:** Expensive — requires domain experts, time, annotation pipelines

**Semi-supervised learning:** Has a small labelled set $\mathcal{D}_L$ and a large unlabelled set $\mathcal{D}_U$.

**Self-supervised learning:** Derives labels automatically from the structure of unlabelled data — no human annotation needed.

---

## 2. Semi-Supervised Learning

### 2.1 Assumptions

Semi-supervised learning requires at least one of these assumptions to hold:

| Assumption | Intuition |
|---|---|
| **Smoothness** | If $x_1 \approx x_2$ in input space, then $y_1 \approx y_2$ |
| **Cluster** | Decision boundary should not cut through dense regions |
| **Manifold** | Data lies on a low-dimensional manifold; labels vary smoothly on it |

### 2.2 Methods

**Pseudo-labelling (Self-training):**
1. Train on $\mathcal{D}_L$
2. Predict labels for $\mathcal{D}_U$ with the current model
3. Add high-confidence predictions to the labelled set
4. Retrain; repeat

Simple but works surprisingly well. Used in modern SSL methods (FixMatch, FlexMatch).

**Label Propagation:**
Build a graph where nodes = all examples, edges weighted by similarity. Propagate labels from labelled to unlabelled nodes via graph diffusion.
$$f = (I - \alpha S)^{-1} y_L \quad \text{where } S = D^{-1/2}WD^{-1/2}$$

**Consistency Regularisation (MixMatch, FixMatch):**
Enforce that the model output is consistent under data augmentations:
$$\mathcal{L}_{unsup} = \mathbb{E}_{x \in \mathcal{D}_U}\left[\|f(x) - f(\text{aug}(x))\|^2\right]$$

**FixMatch (Sohn et al., 2020):** Use strong augmentations on unlabelled data; use pseudo-labels from weakly-augmented version as targets. State-of-the-art semi-supervised classification.

---

## 3. Self-Supervised Learning

Self-supervised learning creates **pretext tasks** — supervised tasks that require no human labels, derived from the data itself. The model learns representations that transfer to real downstream tasks.

### 3.1 Pretext Tasks

| Pretext task | Input | Predicted target |
|---|---|---|
| **Masked language modelling (BERT)** | Text with 15% tokens masked | Masked tokens |
| **Next token prediction (GPT)** | Sequence prefix | Next token |
| **Rotation prediction** | Rotated image | Rotation angle (0°/90°/180°/270°) |
| **Jigsaw puzzle** | Shuffled patches | Correct patch order |
| **Masked autoencoding (MAE)** | Image with 75% patches masked | Masked patches |
| **Contrastive (SimCLR)** | Two views of same image | Should be close in embedding space |

---

## 4. Contrastive Learning

The dominant self-supervised paradigm for vision (2020–present).

### Core Idea
Given a batch of samples, create two views of each sample via data augmentation. The objective:
- **Pull together** (positive pairs): two views of the same sample
- **Push apart** (negative pairs): views of different samples

### SimCLR (Chen et al., 2020)

**Data augmentation:** For each image $x$, sample two augmented views $\tilde{x}_i, \tilde{x}_j$ (random crop + colour jitter + Gaussian blur)

**Projection head:** $z_i = g(f(\tilde{x}_i))$ where $f$ is the backbone (ResNet), $g$ is a 2-layer MLP.

**NT-Xent loss** (Normalised Temperature-scaled Cross-Entropy):
$$\ell(i,j) = -\log\frac{\exp(\text{sim}(z_i,z_j)/\tau)}{\sum_{k \neq i}\exp(\text{sim}(z_i,z_k)/\tau)}$$

where $\text{sim}(u,v) = u^\top v / (\|u\|\|v\|)$ and $\tau$ is the temperature.

Total loss: $\frac{1}{2N}\sum_{k=1}^N[\ell(2k-1, 2k) + \ell(2k, 2k-1)]$

→ Paper: [[Papers/Chen2020_SimCLR|Chen et al. 2020 (SimCLR)]]

### BYOL (Bootstrap Your Own Latent, Grill et al. 2020)
Contrastive learning **without negative pairs** — avoids the need for large batch sizes. Uses an online network and a momentum-updated target network. Online network predicts the target network's representation of a different view.

### MoCo (He et al., 2020)
Uses a momentum encoder and a queue of negative keys to maintain a large dictionary of negatives without requiring large batch sizes.

---

## 5. Masked Autoencoders (MAE)

He et al. (2022) — self-supervised learning via reconstruction of masked patches.

**Procedure:**
1. Randomly mask 75% of image patches
2. Encoder (ViT) processes only the visible 25%
3. Decoder reconstructs the masked patches from the encoder output + mask tokens
4. Loss: MSE on masked patches only

**Why it works:** High masking ratio forces the model to learn global semantic structure (can't just copy neighbours). Pre-training is extremely efficient.

**Results:** After pre-training, a linear probe (single linear layer on top of frozen features) achieves 83.1% top-1 accuracy on ImageNet.

---

## 6. Foundation Models and Pre-training at Scale

Self-supervised learning at scale produces **foundation models** — large models pre-trained on massive unlabelled datasets that transfer to many downstream tasks with little fine-tuning:

| Model | Pretext task | Data | Downstream |
|---|---|---|---|
| BERT | Masked LM + NSP | BooksCorpus + Wikipedia | NLP tasks |
| GPT-4 | Next token prediction | Web-scale text | Any language task |
| CLIP | Image-text contrastive | 400M image-text pairs | Zero-shot vision |
| DINO | Self-distillation (no labels) | ImageNet | Vision features |
| MAE | Masked patch reconstruction | ImageNet | Vision |

**Scaling laws (Kaplan et al., 2020):** Loss scales as a power law with compute, model size, and data — driving the trend toward ever-larger models.

---

## 7. Relevance to Gesture Recognition

- **Small labelled dataset** (100 training samples in user-independent CV) → semi-supervised methods may help
- **Self-supervised pre-training on raw sequences:** Mask portions of gesture trajectories and reconstruct → pre-train BiLSTM representations without labels
- **Data augmentation** (time warping, noise, rotation) as a form of SSL — consistency regularisation enforces augmentation invariance

---

## Connections

- Foundation for → [[Papers/Devlin2018_BERT|BERT]] (masked LM pretext task)
- Contrastive learning paper → [[Papers/Chen2020_SimCLR|SimCLR (Chen 2020)]]
- Unsupervised learning overlap → [[Concepts/Unsupervised_Learning|Unsupervised Learning]]
- Representation learning → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]]
- Autoencoders / VAE → [[Concepts/Generative_Models|Generative Models]], [[Papers/Kingma2013_VAE|VAE]]
- Transfer learning backbone → [[Concepts/Convolutional_Neural_Networks|CNN]] / [[Concepts/Attention_Transformer|Transformers]]
- Reference → [[Papers/Bishop2006_PRML|PRML]], [[Papers/Murphy2012_MLaPP|Murphy]]
