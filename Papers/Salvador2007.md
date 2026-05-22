---
title: "Toward accurate dynamic time warping in linear time and space"
authors: ["Salvador, Stan", "Chan, Philip"]
year: 2007
citekey: "Salvador2007"
doi: "10.3233/IDA-2007-11508"
venue: "Intelligent Data Analysis, 11(5), 561–580"
type: article
tags:
  - literature
  - dtw
  - approximation
  - linear-time
  - scalability
status: read
relevance: medium
created: 2026-05-20
---

# Toward accurate dynamic time warping in linear time and space

> [!info] Metadata
> **Authors:** Stan Salvador, Philip Chan
> **Year:** 2007
> **Venue:** Intelligent Data Analysis, 11(5), 561–580
> **DOI:** [10.3233/IDA-2007-11508](https://doi.org/10.3233/IDA-2007-11508)
> **Citekey:** `Salvador2007`

---

## Abstract

FastDTW achieves $\mathcal{O}(n)$ time and space for DTW via a multilevel coarsening strategy. It recursively downsamples the sequences, solves DTW at coarse resolution, and projects the optimal path upward to constrain the fine-level search. The resulting approximation is highly accurate at a fraction of the computational cost.

---

## Key Contributions

- **FastDTW algorithm:** $\mathcal{O}(n)$ time and space DTW approximation
- Three-step approach: coarsening → projection → refinement
- Empirically near-identical accuracy to exact DTW at $\mathcal{O}(n)$

## Core Methods & Algorithms

1. **Coarsening:** recursively halve sequence resolution
2. **Projection:** use coarse-resolution optimal path to define a narrow search window at the next finer level
3. **Refinement:** solve DTW at each level within the projected window

## Results & Findings

- 10–100× speedup over standard DTW with <1% accuracy loss on benchmark datasets
- Especially valuable for large template libraries or long sequences

## Critical Analysis

**Strengths:**
- Enables DTW-based 1-NN at scale (large N or long sequences)
- Python library `fastdtw` provides a drop-in replacement

**Limitations:**
- Slightly suboptimal path (approximation, not exact)
- For sequences of length 64, standard DTW is already fast enough — FastDTW is more relevant for long sequences

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW baseline]]
- **Role:** `baseline` — efficiency variant
- **Key insight I'm using:** Available as a faster alternative if the 1-NN DTW evaluation becomes a bottleneck in the leave-one-user-out cross-validation. Implemented as `dtw_distance_fast()` in the DTW note.

## Connections

- Builds on → [[Sakoe1978]]
- Benchmarked in → [[Ding2008]]
