---
title: "The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain"
authors: ["Rosenblatt, Frank"]
year: 1958
citekey: "Rosenblatt1958_Perceptron"
doi: "10.1037/h0042519"
venue: "Psychological Review, 65(6), 386–408"
type: article
tags: [literature, perceptron, neural-networks, classification, linear-model, foundational, historical]
status: read
relevance: foundational
created: 2026-05-22
---

# The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain

> [!info] Metadata
> **Authors:** Frank Rosenblatt (Cornell Aeronautical Laboratory)
> **Year:** 1958
> **Venue:** Psychological Review, 65(6), 386–408

---

## Abstract

Introduces the **perceptron** — the first trainable neural network model. A single artificial neuron computes a weighted sum of inputs and fires if the result exceeds a threshold. The perceptron learning rule iteratively updates weights to correctly classify training patterns.

---

## Key Contributions

- **Perceptron model:** $\hat{y} = \text{sign}(w^\top x + b)$ — binary output $\in \{-1, +1\}$
- **Perceptron learning rule:** For misclassified sample $(x_i, y_i)$:
$$w \leftarrow w + \eta \cdot y_i \cdot x_i$$
- **Perceptron Convergence Theorem** (Novikoff, 1962): If the data is linearly separable, the algorithm converges in a finite number of steps
- Showed that a learning machine could discover decision rules from examples without being explicitly programmed

## Historical Significance

The perceptron caused enormous excitement in the late 1950s–60s — news coverage suggested it could learn to "walk, talk, see, write, reproduce itself and be conscious." This led to backlash when Minsky & Papert (1969) proved it cannot learn XOR — triggering the first "AI winter."

The solution: **multi-layer networks** trained with backpropagation → [[Papers/Rumelhart1986_Backprop|Rumelhart et al. 1986]]

## Limitations (Minsky & Papert, 1969)

- Cannot solve non-linearly separable problems (XOR is the canonical example)
- Single layer → only linear decision boundaries
- Led to development of multi-layer perceptrons (MLPs)

## Connections

- Extended to MLP → [[Concepts/Neural_Networks_MLP|Neural Networks & MLP]]
- Overcome by backpropagation → [[Papers/Rumelhart1986_Backprop|Rumelhart et al. 1986]]
- Linear classification → [[Concepts/Kernel_Methods_SVM|SVM]] (perceptron = SVM without margin maximisation)
- Historical context → [[Concepts/Backpropagation|Backpropagation]]
