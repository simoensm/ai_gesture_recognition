---
title: "Deep Learning"
authors: ["Goodfellow, Ian", "Bengio, Yoshua", "Courville, Aaron"]
year: 2016
citekey: "Goodfellow2016"
doi: ""
url: "http://www.deeplearningbook.org"
venue: "MIT Press"
type: book
tags:
  - literature
  - deep-learning
  - textbook
  - rnn
  - lstm
status: reference
relevance: medium
created: 2026-05-20
---

# Deep Learning

> [!info] Metadata
> **Authors:** Ian Goodfellow, Yoshua Bengio, Aaron Courville
> **Year:** 2016
> **Venue:** MIT Press
> **URL:** [deeplearningbook.org](http://www.deeplearningbook.org)
> **Citekey:** `Goodfellow2016`

---

## Abstract

The standard graduate-level textbook on deep learning. Covers feedforward networks, regularization, optimization, convolutional networks, recurrent networks (including LSTM), and unsupervised learning.

---

## Key Chapters for This Project

- **Chapter 10: Sequence Modeling — Recurrent and Recursive Nets** — covers RNN, LSTM, BiLSTM, gradient issues. Direct theoretical background for [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]]
- **Chapter 6: Deep Feedforward Networks** — activation functions, regularization (dropout)
- **Chapter 7: Regularization** — dropout, weight decay — relevant to BiLSTM training

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/04_BiLSTM|BiLSTM (SOTA)]]
- **Role:** `textbook` — theoretical background and citation source for report
- **Key sections:** Chapter 10 for RNN/LSTM theory; used as a reference in "state-of-the-art methods" justification

## Connections

- Covers → [[Hochreiter1997]] (LSTM), [[Schuster1997]] (BiRNN)
