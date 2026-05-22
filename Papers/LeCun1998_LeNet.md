---
title: "Gradient-Based Learning Applied to Document Recognition"
authors: ["LeCun, Yann", "Bottou, Léon", "Bengio, Yoshua", "Haffner, Patrick"]
year: 1998
citekey: "LeCun1998_LeNet"
doi: "10.1109/5.726791"
venue: "Proceedings of the IEEE, 86(11), 2278–2324"
type: article
tags: [literature, cnn, lenet, convolutional-networks, document-recognition, mnist, deep-learning, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Gradient-Based Learning Applied to Document Recognition

> [!info] Metadata
> **Authors:** Yann LeCun, Léon Bottou, Yoshua Bengio, Patrick Haffner (Bell Labs / AT&T)
> **Year:** 1998
> **Venue:** Proceedings of the IEEE, 86(11), 2278–2324

---

## Abstract

Introduces **LeNet-5** — the first successful convolutional neural network trained end-to-end via backpropagation for document recognition (handwritten digits, zip codes). Demonstrates the power of local receptive fields, shared weights, and spatial subsampling, establishing the CNN paradigm that dominates computer vision today.

---

## Key Contributions

- **Convolutional layers:** Local receptive fields + shared weights → translation equivariance + parameter efficiency
- **Subsampling (pooling):** Reduces spatial resolution, increases invariance
- **LeNet-5 architecture:** Conv(6) → Pool → Conv(16) → Pool → FC(120) → FC(84) → Softmax(10)
- **End-to-end training:** Entire pipeline trained jointly via backpropagation — no hand-crafted features
- **Gradient-Based Learning:** Frames recognition as function optimisation from raw pixels to class probabilities

## LeNet-5 Architecture

```
Input (32×32) → C1: 6 feature maps (5×5 conv) → S2: 6@14×14 (avg pool)
→ C3: 16@10×10 (5×5 conv) → S4: 16@5×5 → C5: 120 FC → F6: 84 FC → Output: 10
```

Total parameters: ~60,000 — tiny by modern standards but revolutionary at the time.

## Significance

- First practical deep learning system (despite not yet called "deep learning")
- Deployed commercially in US banking to read millions of cheques daily
- AlexNet (2012) is essentially a wider, deeper LeNet trained on GPUs
- Established: convolution → pooling → FC → softmax as the canonical vision pipeline

## Connections

- Improved by → [[Papers/Krizhevsky2012_AlexNet|AlexNet (Krizhevsky 2012)]]
- Further improved by → [[Papers/He2016_ResNet|ResNet (He 2016)]]
- Architecture details → [[Concepts/Convolutional_Neural_Networks|Convolutional Neural Networks]]
- Training uses → [[Concepts/Backpropagation|Backpropagation]]
- Same author (LeCun) as → foundational work on weight initialisation, sparse coding
