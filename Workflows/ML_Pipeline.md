---
title: "End-to-End Machine Learning Pipeline"
tags: [workflow, pipeline, ml, supervised-learning, practical]
created: 2026-05-22
---

# End-to-End Machine Learning Pipeline

> A structured overview of every stage from raw data to deployed model. Each stage links to dedicated notes for deeper detail.

---

## Overview

```
Raw Data
    │
    ▼
01. Problem Framing
    │
    ▼
02. Data Collection & Understanding
    │
    ▼
03. Data Preprocessing & Feature Engineering
    │
    ▼
04. Model Selection & Training
    │
    ▼
05. Evaluation & Error Analysis
    │
    ▼
06. Hyperparameter Tuning
    │
    ▼
07. Final Evaluation on Test Set
    │
    ▼
08. Deployment & Monitoring
```

---

## Stage 1 — Problem Framing

Before writing any code, define:
- **Task type:** Classification / Regression / Clustering / Generation / Ranking
- **Input-output specification:** What goes in, what comes out, what format?
- **Success metric:** What KPI or loss function defines "good"?
- **Baseline:** What does the simplest possible model achieve?
- **Constraints:** Latency, memory, interpretability, privacy?

For the gesture recognition project: multiclass classification (10 classes), evaluation metric = accuracy (balanced classes), baseline = DTW 1-NN.

---

## Stage 2 — Data Collection & Understanding

- **Source and size:** How many samples? Class balance? Collection protocol?
- **Exploratory Data Analysis (EDA):** Plot distributions, class frequencies, sequence lengths, outliers
- **Data quality:** Missing values, duplicate samples, labelling errors, sensor drift
- **Train / Validation / Test split:** Define before touching data. Respect temporal or user boundaries.

> For the Huang et al. dataset: 10 users × 10 classes × 10 reps = 1,000 sequences. User-independent CV: leave-one-user-out (10 folds). User-dependent CV: leave-one-sample-out.

See → [[Concepts/Cross_Validation|Cross-Validation Protocols]]

---

## Stage 3 — Data Preprocessing & Feature Engineering

See → [[Workflows/Data_Preprocessing|Data Preprocessing in Detail]]

Key steps:
1. **Cleaning:** Remove outliers, fix labels, handle missing frames
2. **Normalisation:** Zero-mean unit-variance per feature / global min-max
3. **Resampling:** Resample all sequences to fixed length $T$ (required by $3 Recognizer)
4. **Feature extraction (classical):** Hand-crafted features (velocity, curvature, angles); only for non-end-to-end methods
5. **Augmentation:** Time warping, Gaussian noise, rotation jitter (especially for BiLSTM training)

---

## Stage 4 — Model Selection & Training

### Selection criteria
- Dataset size → small data favours SVM/RF; large data favours deep learning
- Sequence data → DTW or RNN/LSTM/Transformer
- Interpretability requirements → decision trees / linear models

### Training protocol
- **Loss function:** Cross-entropy (multiclass), MSE (regression), CTC (sequence-to-sequence)
- **Optimiser:** Adam (default), AdamW (transformers), SGD+momentum (large-batch)
- **Regularisation:** L2 weight decay, dropout, early stopping, data augmentation
- **Learning rate:** Start with `1e-3`, use warm-up + cosine decay for transformers

See → [[Concepts/Backpropagation|Backpropagation]] | [[Concepts/Bias_Variance_Regularisation|Regularisation]] | [[Foundations/Optimisation_Theory|Optimisation Theory]]

---

## Stage 5 — Evaluation & Error Analysis

See → [[Workflows/Model_Evaluation|Model Evaluation in Detail]]

- **Confusion matrix:** Which classes are confused with which?
- **Per-class accuracy:** Are failures concentrated in a few classes?
- **Error modes:** Systematic (model bias) vs. random (high variance / ambiguous samples)
- **Learning curves:** Training vs. validation loss over epochs → diagnose over/underfitting

---

## Stage 6 — Hyperparameter Tuning

See → [[Workflows/Hyperparameter_Tuning|Hyperparameter Tuning in Detail]]

- **Search strategy:** Grid (small space), random (larger space), Bayesian (expensive evaluations)
- **What to tune first:** Learning rate > architecture width/depth > regularisation strength
- **Validation set** must be held out during tuning — never tune on the test set

---

## Stage 7 — Final Test Evaluation

- Run exactly **once** on the held-out test set
- Report: accuracy, macro-F1, confusion matrix
- Use appropriate statistical tests for significance (McNemar's test for classifiers)
- Compare against published baselines using the **same split and metric**

---

## Stage 8 — Deployment & Monitoring

- Export model (ONNX, TorchScript, SavedModel)
- Monitor for **data drift**: distribution of incoming gestures may shift over time
- Logging & alerting: track prediction confidence, latency, error rate
- Retrain periodically with new data

---

## Connections

- Data preprocessing → [[Workflows/Data_Preprocessing|Data Preprocessing]]
- Evaluation → [[Workflows/Model_Evaluation|Model Evaluation]]
- Hyperparameter tuning → [[Workflows/Hyperparameter_Tuning|Hyperparameter Tuning]]
- Gesture project applied pipeline → [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition MOC]]
