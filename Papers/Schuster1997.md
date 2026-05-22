---
title: "Bidirectional recurrent neural networks"
authors: ["Schuster, Mike", "Paliwal, Kuldip K."]
year: 1997
citekey: "Schuster1997"
doi: "10.1109/78.650093"
venue: "IEEE Transactions on Signal Processing, 45(11), 2673–2681"
type: article
tags:
  - literature
  - bilstm
  - rnn
  - bidirectional
  - sequence-modeling
  - sota
status: read
relevance: high
created: 2026-05-20
---

# Bidirectional recurrent neural networks

> [!info] Metadata
> **Authors:** Mike Schuster, Kuldip K. Paliwal
> **Year:** 1997
> **Venue:** IEEE Transactions on Signal Processing, 45(11), 2673–2681
> **DOI:** [10.1109/78.650093](https://doi.org/10.1109/78.650093)
> **Citekey:** `Schuster1997`

---

## Abstract

Introduces the bidirectional RNN architecture: two independent RNNs process the same sequence in forward and backward directions, and their hidden states are concatenated at each time step. This allows the network to condition its representation of each time step on both past and future context.

---

## Key Contributions

- **Bidirectional processing:** forward LSTM ($\overrightarrow{h}_t$) + backward LSTM ($\overleftarrow{h}_t$)
- Output: $h_t = [\overrightarrow{h}_t;\ \overleftarrow{h}_t] \in \mathbb{R}^{2H}$
- Showed that bidirectional context improves accuracy on speech recognition
- Architecture now standard in NLP, ASR, and gesture recognition

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/04_BiLSTM|BiLSTM (SOTA)]]
- **Role:** `sota` — defines the bidirectional architecture we use
- **Key insight I'm using:** In gesture recognition, the end of a gesture (hand returning to rest position) is highly discriminative. Bidirectionality lets the model use future context to interpret earlier frames — a strong inductive bias for gesture sequences.

## Connections

- Combined with → [[Hochreiter1997]] to form BiLSTM
- Empirically validated for gesture classification → [[Graves2005_BiLSTM]]
- Applied to skeleton data → [[Zhang2021_SkeletonBiLSTM]]
