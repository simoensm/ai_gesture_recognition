---
title: "Deep Residual Learning for Image Recognition (ResNet)"
authors: ["He, Kaiming", "Zhang, Xiangyu", "Ren, Shaoqing", "Sun, Jian"]
year: 2016
citekey: "He2016_ResNet"
doi: "10.1109/CVPR.2016.90"
venue: "CVPR 2016, pp. 770–778"
type: conference
tags: [literature, resnet, residual-connections, deep-learning, cnn, skip-connections, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Deep Residual Learning for Image Recognition (ResNet)

> [!info] Metadata
> **Authors:** Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun
> **Year:** 2016
> **Venue:** CVPR 2016, pp. 770–778
> **DOI:** [10.1109/CVPR.2016.90](https://doi.org/10.1109/CVPR.2016.90)

---

## Abstract

ResNet won ILSVRC 2015 with a 3.57% top-5 error using a 152-layer network — deeper than anything previously trainable. The key innovation: **residual (skip) connections** that let gradients bypass layers, solving the degradation problem in very deep networks.

---

## Key Contributions

- **Residual block:** $y = F(x, W) + x$ — the identity shortcut allows gradients to flow directly
- Solved the "degradation problem": deeper networks were getting worse, not better — now resolved by learning residuals
- Enabled training of networks with 50, 101, 152, and even 1000+ layers
- Standard building block for virtually all modern vision architectures
- Bottleneck block: 1×1 → 3×3 → 1×1 convolutions to reduce computation in deep networks

## Intuition

The network learns the **residual** $F(x) = H(x) - x$ rather than $H(x)$ directly. If the optimal mapping is close to the identity, it's much easier to push $F \to 0$ than to parameterise the identity mapping from scratch.

## Results

- ILSVRC 2015: 3.57% top-5 error (152-layer network)
- Won ImageNet detection, localisation, and segmentation challenges

## Connections

- Extension of → [[Papers/Krizhevsky2012_AlexNet|AlexNet]]
- Residual connections solve → [[Concepts/Backpropagation|vanishing gradient problem]]
- Architecture → [[Concepts/Convolutional_Neural_Networks|CNN]]
- Concept generalised to → [[Concepts/Attention_Transformer|Transformers]] (residuals in every block)
