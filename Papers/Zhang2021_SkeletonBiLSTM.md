---
title: "3D Skeletal Joints-Based Hand Gesture Spotting and Classification"
authors: ["Zhang, Junwei", "Zhou, Wengang", "Xie, Chang", "Pu, Jianyun", "Li, Houqiang"]
year: 2021
citekey: "Zhang2021_SkeletonBiLSTM"
doi: "10.3390/app11104689"
venue: "Applied Sciences, 11(10), 4689"
type: article
tags:
  - literature
  - bilstm
  - skeleton
  - hand-gesture
  - 3d
  - classification
  - sota
status: read
relevance: high
created: 2026-05-20
---

# 3D Skeletal Joints-Based Hand Gesture Spotting and Classification

> [!info] Metadata
> **Authors:** Junwei Zhang, Wengang Zhou, Chang Xie, Jianyun Pu, Houqiang Li
> **Year:** 2021
> **Venue:** Applied Sciences, 11(10), 4689
> **DOI:** [10.3390/app11104689](https://doi.org/10.3390/app11104689)
> **Citekey:** `Zhang2021_SkeletonBiLSTM`

---

## Abstract

Proposes a BiLSTM-based pipeline for hand gesture spotting (locating gesture boundaries in a continuous stream) and classification using 3D skeletal joint data. Directly relevant to our project as it uses the same data modality: 3D joint positions over time, classified into gesture categories.

---

## Key Contributions

- End-to-end BiLSTM pipeline for 3D skeleton gesture recognition
- Input: 3D joint positions (same as our project's data format)
- Demonstrates that BiLSTM handles the joint-position time series naturally
- Joint attention mechanism to weight important joints

## Results & Findings

- Strong classification accuracy on skeleton-based gesture datasets
- BiLSTM outperforms unidirectional LSTM, validating bidirectionality for gesture classification

## Critical Analysis

**Strengths:**
- Directly applicable to our project: same input modality (3D joint positions)
- Provides reference accuracy numbers for BiLSTM on skeleton gesture data
- Well-engineered pipeline with velocity features and attention pooling

**Limitations:**
- Uses larger datasets than our 1000-sample project dataset
- May overfit on our small dataset without heavy regularization

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/04_BiLSTM|BiLSTM (SOTA)]]
- **Role:** `sota` — closest reference to our exact setup (3D skeleton + BiLSTM + classification)
- **Key insight I'm using:** Our `BiLSTMGestureClassifier` architecture is inspired by this paper. The velocity feature augmentation (`add_velocity_features()`) is validated here.

## Connections

- Builds on → [[Hochreiter1997]], [[Schuster1997]], [[Graves2005_BiLSTM]]
- See also → [[Zhu2019]] (efficiency), [[Huang2019]] (dataset)
