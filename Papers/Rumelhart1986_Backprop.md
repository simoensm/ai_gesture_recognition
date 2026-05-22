---
title: "Learning representations by back-propagating errors"
authors: ["Rumelhart, David E.", "Hinton, Geoffrey E.", "Williams, Ronald J."]
year: 1986
citekey: "Rumelhart1986_Backprop"
doi: "10.1038/323533a0"
venue: "Nature, 323, 533–536"
type: article
tags: [literature, backpropagation, neural-networks, deep-learning, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Learning representations by back-propagating errors

> [!info] Metadata
> **Authors:** David E. Rumelhart, Geoffrey E. Hinton, Ronald J. Williams
> **Year:** 1986
> **Venue:** Nature, 323, 533–536
> **DOI:** [10.1038/323533a0](https://doi.org/10.1038/323533a0)

---

## Abstract

Demonstrates that the backpropagation algorithm can train multi-layer neural networks to learn internal (hidden) representations of data. This was the breakthrough that made multi-layer networks trainable, launching the connectionist movement and ultimately modern deep learning.

---

## Key Contributions

- Formalised the backpropagation algorithm as an efficient application of the chain rule to compute gradients through multi-layer networks
- Demonstrated that hidden layers develop useful internal representations (distributed representations)
- Showed networks can learn the XOR function (which single-layer perceptrons cannot)

## Core Algorithm

1. Forward pass: compute activations layer by layer
2. Compute loss
3. Backward pass: propagate $\delta^{(l)} = (W^{(l+1)})^\top \delta^{(l+1)} \odot \sigma'(z^{(l)})$
4. Compute weight gradients: $\partial L/\partial W^{(l)} = \delta^{(l)} (a^{(l-1)})^\top$
5. Update: $W \leftarrow W - \eta \partial L/\partial W$

## Connections

- Algorithm explained in → [[Concepts/Backpropagation|Backpropagation]]
- Foundation of → [[Concepts/Neural_Networks_MLP|Neural Networks]]
- All modern DL frameworks implement this
