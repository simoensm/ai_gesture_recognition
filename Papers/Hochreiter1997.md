---
title: "Long short-term memory"
authors: ["Hochreiter, Sepp", "Schmidhuber, Jürgen"]
year: 1997
citekey: "Hochreiter1997"
doi: "10.1162/neco.1997.9.8.1735"
venue: "Neural Computation, 9(8), 1735–1780"
type: article
tags:
  - literature
  - lstm
  - rnn
  - deep-learning
  - vanishing-gradient
  - sequence-modeling
  - sota
status: read
relevance: high
created: 2026-05-20
---

# Long short-term memory

> [!info] Metadata
> **Authors:** Sepp Hochreiter, Jürgen Schmidhuber
> **Year:** 1997
> **Venue:** Neural Computation, 9(8), 1735–1780
> **DOI:** [10.1162/neco.1997.9.8.1735](https://doi.org/10.1162/neco.1997.9.8.1735)
> **Citekey:** `Hochreiter1997`

---

## Abstract

Introduces the Long Short-Term Memory (LSTM) architecture, designed to overcome the vanishing gradient problem in standard RNNs. The LSTM uses a gated cell state that flows through time with additive updates — allowing gradients to flow through hundreds of time steps without vanishing. This paper is one of the most cited in all of deep learning.

---

## Key Contributions

- Identified and formalized the **vanishing gradient problem** in RNNs (Hochreiter, 1991; this paper)
- Introduced the **cell state** $C_t$ as a "memory highway" with only multiplicative (element-wise) gates
- Three gate types: **forget gate** $f_t$, **input gate** $i_t$, **output gate** $o_t$
- Gated cell update: $C_t = f_t \odot C_{t-1} + i_t \odot \tilde{C}_t$

## Core Methods & Algorithms

**Forget gate:** $f_t = \sigma(W_f [h_{t-1}, x_t] + b_f)$
**Input gate:** $i_t = \sigma(W_i [h_{t-1}, x_t] + b_i)$, $\tilde{C}_t = \tanh(W_C [h_{t-1}, x_t] + b_C)$
**Cell update:** $C_t = f_t \odot C_{t-1} + i_t \odot \tilde{C}_t$
**Output gate:** $o_t = \sigma(W_o [h_{t-1}, x_t] + b_o)$, $h_t = o_t \odot \tanh(C_t)$

Key property: $\frac{\partial C_t}{\partial C_{t-1}} = f_t$ (learned, not fixed) → no vanishing gradient.

## Results & Findings

- Outperforms standard RNNs on long-range dependency tasks
- Gradient can flow through 1000+ time steps without vanishing

## Critical Analysis

**Strengths:**
- Elegantly solves the fundamental RNN limitation
- Highly general — applicable to any sequential data
- Well-supported in all major frameworks (PyTorch: `nn.LSTM`)

**Limitations:**
- Computationally expensive vs. simpler RNNs (4x parameter count per layer)
- Replaced by Transformers for many tasks, but still optimal for short sequences with limited data

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/04_BiLSTM|BiLSTM (SOTA)]]
- **Role:** `sota` — foundational architecture
- **Key insight I'm using:** The `nn.LSTM(..., bidirectional=True)` in PyTorch implements the BiLSTM stack. The forget gate (introduced in [[Gers2000]]) is on by default in PyTorch.

## Connections

- Forget gate added in → [[Gers2000]]
- Bidirectional extension → [[Schuster1997]]
- Applied to BiLSTM for classification → [[Graves2005_BiLSTM]]
- Applied to skeleton gestures → [[Zhang2021_SkeletonBiLSTM]]
- Course concept note → [[Course - Sequence Modeling/Hidden Markov Model (HMM)|HMM]] (related sequential model)
