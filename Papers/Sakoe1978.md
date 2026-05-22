---
title: "Dynamic programming algorithm optimization for spoken word recognition"
authors: ["Sakoe, Hiroaki", "Chiba, Seibi"]
year: 1978
citekey: "Sakoe1978"
doi: "10.1109/TASSP.1978.1163055"
venue: "IEEE Transactions on Acoustics, Speech, and Signal Processing"
type: article
tags:
  - literature
  - dtw
  - dynamic-programming
  - time-series
  - baseline
  - speech-recognition
status: read
relevance: high
created: 2026-05-20
---

# Dynamic programming algorithm optimization for spoken word recognition

> [!info] Metadata
> **Authors:** Hiroaki Sakoe, Seibi Chiba
> **Year:** 1978
> **Venue:** IEEE Transactions on Acoustics, Speech, and Signal Processing, 26(1), 43–49
> **DOI:** [10.1109/TASSP.1978.1163055](https://doi.org/10.1109/TASSP.1978.1163055)
> **Citekey:** `Sakoe1978`

---

## Abstract

This paper presents a dynamic programming (DP) algorithm for spoken word recognition that optimally aligns two speech patterns that may differ in speaking rate. The authors introduce the concept of the **Sakoe-Chiba band** — a diagonal constraint on the warping path — and a symmetric DP formulation that outperforms asymmetric alternatives. They demonstrate that constrained DTW significantly reduces computation while improving recognition accuracy.

---

## Key Contributions

- Formalized the DTW recurrence relation for sequence alignment: $D(i,j) = C(i,j) + \min\{D(i-1,j),\ D(i,j-1),\ D(i-1,j-1)\}$
- Introduced the **Sakoe-Chiba band** constraint: $|i - j| \leq r$, reducing complexity from $\mathcal{O}(nm)$ to $\mathcal{O}(rn)$
- Showed that **symmetric** DTW (equal weighting of time steps) outperforms asymmetric formulations for speech
- Introduced **slope constraints** to prevent degenerate warping paths

## Core Methods & Algorithms

- Dynamic Programming on a cost matrix $C(i,j) = \delta(x_i, y_j)$
- Global path constraint (Sakoe-Chiba band radius $r$)
- Symmetric step pattern: allows $(1,0)$, $(0,1)$, $(1,1)$ steps

## Results & Findings

- DTW with the Sakoe-Chiba band achieves competitive accuracy for spoken word recognition
- The symmetric formulation consistently outperforms asymmetric DTW
- Slope constraints (limiting path steepness) further improve accuracy

## Critical Analysis

**Strengths:**
- Elegant mathematical formulation that became the standard
- Computationally tractable with the band constraint
- Surprisingly robust: still a hard-to-beat baseline 45 years later

**Limitations:**
- Originally designed for 1D speech signals; 3D gesture application requires $\delta$ to be a multivariate distance
- The warping window radius $r$ is a manual hyperparameter
- Sensitive to outlier frames (a single noisy sample affects the entire path)

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW baseline]]
- **Role:** `baseline` — foundational algorithm
- **Key insight I'm using:** The Sakoe-Chiba band (10% of sequence length) is our default warping window. The symmetric DP step pattern is what `tslearn` implements by default.

## Connections

- Directly implemented in [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|01_DTW_Dynamic_Time_Warping]]
- Extended by derivative variant → [[Keogh2001_DDTW]]
- Approximated at linear time → [[Salvador2007]]
- Benchmarked against other methods in → [[Ding2008]]
- Concept note: [[Concepts/Dynamic_Programming|Dynamic Programming]]
- Concept note: [[Concepts/kNN_Classifier|k-NN Classifier]]

---

## My Notes

The warping window radius is expressed as a fraction in `tslearn`: `sakoe_chiba_radius = int(0.10 * target_len)`. For `target_len=64`, this gives radius 6. The paper recommends ~10% as a good default, matching our ablation choices.
