---
title: "Bidirectional LSTM Networks for Improved Phoneme Classification and Recognition"
authors: ["Graves, Alex", "Fernández, Santiago", "Schmidhuber, Jürgen"]
year: 2005
citekey: "Graves2005_BiLSTM"
doi: "10.1007/11550907_126"
venue: "ICANN 2005, Springer, pp. 799–804"
type: conference
tags:
  - literature
  - bilstm
  - rnn
  - sequence-classification
  - phoneme-recognition
  - sota
status: read
relevance: high
created: 2026-05-20
---

# Bidirectional LSTM Networks for Improved Phoneme Classification and Recognition

> [!info] Metadata
> **Authors:** Alex Graves, Santiago Fernández, Jürgen Schmidhuber
> **Year:** 2005
> **Venue:** International Conference on Artificial Neural Networks (ICANN), Springer, pp. 799–804
> **DOI:** [10.1007/11550907_126](https://doi.org/10.1007/11550907_126)
> **Citekey:** `Graves2005_BiLSTM`

---

## Abstract

Empirically demonstrates that BiLSTM outperforms forward-only LSTM for phoneme classification and recognition on speech data. This paper provides the first strong experimental evidence that bidirectionality provides consistent gains over unidirectional LSTM on sequential classification tasks.

---

## Key Contributions

- First empirical demonstration that BiLSTM > LSTM for sequential classification
- Established BiLSTM as the standard architecture for sequence labeling tasks
- Demonstrated that backward context is essential for tasks where the "resolution" of ambiguity depends on future observations

## Results & Findings

- BiLSTM consistently outperforms forward LSTM on phoneme tasks
- The improvement is most pronounced when future context resolves ambiguity in the current frame

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/04_BiLSTM|BiLSTM (SOTA)]]
- **Role:** `sota` — key empirical reference
- **Key insight I'm using:** The architecture in `04_BiLSTM` (`BiLSTMGestureClassifier`) follows the architecture validated here. Good citation for the "Why BiLSTM?" section of the report.

## Connections

- Combines → [[Hochreiter1997]] (LSTM) + [[Schuster1997]] (BiRNN)
- Applied to gestures → [[Zhang2021_SkeletonBiLSTM]]
