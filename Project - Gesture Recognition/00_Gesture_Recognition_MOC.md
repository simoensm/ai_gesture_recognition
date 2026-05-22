---
title: "Gesture Recognition — Project MOC"
course: MLSM2154
tags:
  - MOC
  - gesture-recognition
  - supervised-learning
  - sequence-classification
  - ablation-study
  - project
created: 2026-05-22
---

# Gesture Recognition — Ablation Study MOC

> **Course:** MLSM2154  
> **Assignment:** [[Project Assignment|Project Assignment]]  
> **Dataset:** [[Papers/Huang2019|Huang et al. (2019)]] — 10 users × 10 classes × 10 reps = 1,000 sequences  
> **Goal:** Compare baseline and SOTA methods on 3D hand gesture recognition  
> **Papers index:** [[Papers/00_Papers_MOC|Papers MOC]]  
> **Master knowledge graph:** [[00_AI_Knowledge_Graph_MOC|AI Knowledge Graph MOC]]  
> **Evaluation:** [[Concepts/Cross_Validation|Cross-Validation Protocol]] · [[Workflows/Model_Evaluation|Model Evaluation]]  
> **Pipeline:** [[Workflows/ML_Pipeline|ML Pipeline]] · [[Workflows/Data_Preprocessing|Data Preprocessing]]

---

## Methods Overview

| Method | Role | Approach | Key Paper |
|---|---|---|---|
| [[01_DTW_Dynamic_Time_Warping]] | Baseline | Elastic time-series alignment | [[Papers/Sakoe1978\|Sakoe & Chiba (1978)]] |
| [[02_Edit_Distance]] | Baseline | Quantize → string edit ops | [[Papers/Levenshtein1966\|Levenshtein (1966)]], [[Papers/Nyirarugira2014\|Nyirarugira (2014)]] |
| [[03_Three_Cent_Recognizer]] | SOTA (template) | 3D geometric template matching | [[Papers/Kratz2010\|Kratz & Rohs (2010)]] |
| [[04_BiLSTM]] | SOTA (deep learning) | Bidirectional RNN sequence classifier | [[Papers/Hochreiter1997\|Hochreiter & Schmidhuber (1997)]] |

---

## Shared Ablation Protocol

- **CV modes:** user-dependent (UD) and user-independent (UI)
- **Metric:** recognition accuracy (mean ± std across folds), F1-macro
- **Fixed folds:** shuffle once, save splits, reuse across all methods
- **One-factor-at-a-time:** vary one parameter, fix all others at baseline

## Key Parameters Per Method

### [[01_DTW_Dynamic_Time_Warping]]
- Resampling length: 32 / **64** / 128
- Sakoe-Chiba window: None / 5% / **10%** / 20% / 30%
- Derivative DTW: off / **on**
- Normalize: off / **on**

### [[02_Edit_Distance]]
- Codebook size: 8 / **16** / 32 / 64
- Resampling length: 32 / **64** / 128
- Weighted edit distance: **False** / True
- ins/del cost: 0.5 / **1.0** / 2.0

### [[03_Three_Cent_Recognizer]]
- Resampling points N: 32 / **64** / 128
- Rotation method: **kabsch** / gss / none
- PCA alignment: **True** / False
- Scale to unit cube: **True** / False

### [[04_BiLSTM]]
- Hidden size: 64 / **128** / 256
- Layers: 1 / **2** / 3
- Dropout: 0.0 / 0.2 / **0.3** / 0.5
- Input features: raw / **+velocity** / +velocity+accel
- Pooling: last / **mean** / attention

---

## Concept Notes

### Project-Specific
- [[Concepts/Dynamic_Programming|Dynamic Programming]] — shared algorithmic backbone of DTW, Edit Distance, and LCS
- [[Concepts/Longest_Common_Subsequence|Longest Common Subsequence (LCS)]] — baseline alternative to edit distance
- [[Concepts/Vector_Quantization|Vector Quantization]] — how continuous gestures become strings for Edit Distance / LCS
- [[Concepts/kNN_Classifier|k-NN Classifier]] — all baseline methods use 1-NN classification
- [[Concepts/Cross_Validation|Cross-Validation Protocol]] — user-independent (leave-one-user-out) and user-dependent (leave-one-sample-out)
- [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]] — optimal rotation for $3 Recognizer

### ML Foundations (Background)
- [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]] — ERM, loss functions, generalisation bounds
- [[Concepts/Clustering_EM|Clustering & EM]] — k-means for VQ codebook construction
- [[Concepts/Dimensionality_Reduction|Dimensionality Reduction]] — PCA preprocessing option
- [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]] — dropout, weight decay, early stopping for BiLSTM
- [[Concepts/Backpropagation|Backpropagation]] — training the BiLSTM
- [[Concepts/Neural_Networks_MLP|Neural Networks]] — foundation for BiLSTM architecture

### Workflows
- [[Workflows/Data_Preprocessing|Data Preprocessing]] — standardisation, resampling, feature engineering, augmentation
- [[Workflows/Model_Evaluation|Model Evaluation]] — accuracy, confusion matrix, McNemar's test
- [[Workflows/Hyperparameter_Tuning|Hyperparameter Tuning]] — grid/random/Bayesian search for BiLSTM

---

## Report Checklist

- [ ] Introduction (problem, dataset, methods overview)
- [ ] Theoretical explanation: DTW with equations → [[01_DTW_Dynamic_Time_Warping]]
- [ ] Theoretical explanation: Edit Distance + VQ → [[02_Edit_Distance]]
- [ ] Theoretical explanation: $3 + Kabsch → [[03_Three_Cent_Recognizer]]
- [ ] Theoretical explanation: BiLSTM gates → [[04_BiLSTM]]
- [ ] Results table: mean ± std per method × {UD, UI}
- [ ] Confusion matrix for best method
- [ ] Statistical hypothesis test (user-independent setting)
- [ ] Conclusion

---

## References

All BibTeX entries are in `gesture_recognition.bib`.  
Full paper notes: [[Papers/00_Papers_MOC|Papers MOC]]  
Import into Zotero: **File → Import → gesture_recognition.bib**  
Then import into Obsidian: `Ctrl+P` → **Zotero Integration: Import Notes**
