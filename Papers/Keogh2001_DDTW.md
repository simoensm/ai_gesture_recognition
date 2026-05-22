---
title: "Derivative Dynamic Time Warping"
authors: ["Keogh, Eamonn J.", "Pazzani, Michael J."]
year: 2001
citekey: "Keogh2001_DDTW"
doi: "10.1137/1.9781611972719.1"
venue: "SIAM International Conference on Data Mining (SDM)"
type: conference
tags:
  - literature
  - dtw
  - derivative-dtw
  - time-series
  - feature-engineering
  - baseline
status: read
relevance: high
created: 2026-05-20
---

# Derivative Dynamic Time Warping

> [!info] Metadata
> **Authors:** Eamonn J. Keogh, Michael J. Pazzani
> **Year:** 2001
> **Venue:** SIAM International Conference on Data Mining (SDM), pp. 1–11
> **DOI:** [10.1137/1.9781611972719.1](https://doi.org/10.1137/1.9781611972719.1)
> **Citekey:** `Keogh2001_DDTW`

---

## Abstract

Standard DTW compares raw amplitude values, which can lead to counter-intuitive alignments. This paper proposes Derivative DTW (DDTW), which replaces each raw observation with an estimate of its first derivative before applying DTW. This shifts the similarity measure from amplitude-matching to shape-matching, which is more discriminative for many time-series classification tasks.

---

## Key Contributions

- Identified a fundamental failure mode of standard DTW: pathological alignments caused by comparing absolute values instead of local shape
- Proposed replacing each point $x_i$ with its estimated first derivative:
$$\hat{x}_i = \frac{(x_i - x_{i-1}) + \frac{(x_{i+1} - x_{i-1})}{2}}{2}$$
- Demonstrated DDTW outperforms standard DTW on multiple UCR datasets

## Core Methods & Algorithms

- Feature transformation: raw → derivative estimate (central differences for interior, one-sided at boundaries)
- Same DTW recurrence applied on transformed sequences
- Makes the similarity measure **translation-invariant** in amplitude

## Results & Findings

- DDTW reduces error rate vs standard DTW on most benchmark datasets tested
- Especially beneficial when gestures differ in shape but not in absolute position

## Critical Analysis

**Strengths:**
- Simple drop-in replacement: just preprocess before calling DTW
- Makes the measure invariant to gesture starting position (offset) — crucial for user-independent evaluation
- No extra parameters beyond the standard DTW window

**Limitations:**
- Loses absolute amplitude information — may hurt if amplitude differences are discriminative
- Derivative estimation is noisy at sequence boundaries
- Doubles sensitivity to frame-level noise (since derivatives amplify high-frequency components)

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW baseline]]
- **Role:** `baseline` — key ablation variant
- **Key insight I'm using:** `use_derivatives=True` in our preprocessing pipeline implements DDTW. The formula matches what is in `compute_derivatives()` in the DTW implementation.

## Connections

- Builds on → [[Sakoe1978]]
- Benchmarked in → [[Ding2008]]
- Concept note: [[Concepts/Dynamic_Programming|Dynamic Programming]]

---

## My Notes

In Python: `DDTW` = apply `compute_derivatives()` to the sequence, then feed to standard DTW. The `use_derivatives` ablation in `01_DTW_Dynamic_Time_Warping` covers this directly.
