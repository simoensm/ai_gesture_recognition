---
title: "Modified Levenshtein distance for real-time gesture recognition"
authors: ["Nyirarugira, Clement", "Choi, Tae-Sun"]
year: 2014
citekey: "Nyirarugira2014"
doi: "10.1109/ICASSP.2014.6854524"
venue: "IEEE ICASSP 2014, pp. 4851–4855"
type: conference
tags:
  - literature
  - edit-distance
  - gesture-recognition
  - real-time
  - temporal-variability
  - baseline
status: read
relevance: high
created: 2026-05-20
---

# Modified Levenshtein distance for real-time gesture recognition

> [!info] Metadata
> **Authors:** Clement Nyirarugira, Tae-Sun Choi
> **Year:** 2014
> **Venue:** IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), pp. 4851–4855
> **DOI:** [10.1109/ICASSP.2014.6854524](https://doi.org/10.1109/ICASSP.2014.6854524)
> **Citekey:** `Nyirarugira2014`

---

## Abstract

This paper directly addresses the limitations of standard edit distance for gesture recognition — specifically, its sensitivity to temporal variability in gesture execution speed. The authors propose a modified Levenshtein distance that allows adjacent repeated symbols to match at zero cost, achieving 93.9% real-time recognition accuracy.

---

## Key Contributions

- Identified the key failure mode of standard edit distance on gesture data: penalizing natural speed variation
- Proposed zero-cost matching for repeated adjacent symbols: $\text{cost}(s_i, t_j) = 0$ if $s_i = t_j$
- Applied variable insertion/deletion costs to further handle speed variation
- Demonstrated 93.9% accuracy in real-time gesture recognition

## Core Methods & Algorithms

- Standard Wagner-Fischer DP as base
- Modified substitution cost: uniform 0 or 1
- The key insight: gestures that are "stretched in time" produce repeated adjacent symbols when quantized; allowing zero-cost matches handles this implicitly

## Results & Findings

- 93.9% recognition accuracy in real-time gesture system
- Significantly outperforms unmodified Levenshtein on variable-speed gestures

## Critical Analysis

**Strengths:**
- Handles temporal variability without explicit resampling (unlike DTW or $3)
- Very fast — no warping, just DP on short strings

**Limitations:**
- Still requires a codebook (vector quantization step)
- Performance depends heavily on codebook quality
- Real-time mode uses single exemplar per class — less robust than multi-template KNN

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance baseline]]
- **Role:** `baseline` — the primary edit distance reference for gesture recognition
- **Key insight I'm using:** Justification that edit distance is a legitimate, competitive baseline for gesture recognition. The 93.9% accuracy number sets an expectation for what our edit distance baseline can achieve.

## Connections

- Builds on → [[Levenshtein1966]], [[Wagner1974]]
- Compared against → [[Gavrila1999]] (survey context)
- Concept note: [[Concepts/Vector_Quantization|Vector Quantization]]
