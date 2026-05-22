---
title: "A Comparative Study of Several Dynamic Time-Warping Algorithms for Connected-Word Recognition"
authors: ["Myers, Cory S.", "Rabiner, Lawrence R."]
year: 1981
citekey: "Myers1981_DTW"
doi: "10.1002/j.1538-7305.1981.tb00272.x"
venue: "The Bell System Technical Journal, 60(7), 1389–1409"
type: article
tags: [literature, dtw, dynamic-time-warping, speech-recognition, path-constraints, comparative-study, connected-word]
status: read
relevance: high
created: 2026-05-22
---

# A Comparative Study of Several Dynamic Time-Warping Algorithms for Connected-Word Recognition

> [!info] Metadata
> **Authors:** Cory S. Myers, Lawrence R. Rabiner (Bell Laboratories)
> **Year:** 1981
> **Venue:** The Bell System Technical Journal, 60(7), 1389–1409

---

## Abstract

A comprehensive empirical and theoretical comparison of DTW algorithms for connected-word (continuous) speech recognition. Evaluates different **path step patterns**, **slope constraints**, **endpoint handling**, and **normalisation** strategies — the most thorough study of DTW design choices to appear in the literature. Many of the "standard" DTW configurations used today trace back to the recommendations in this paper.

---

## Key Contributions

### Classification of DTW Path Patterns

Myers & Rabiner systematically classify DTW algorithms by their **allowable step patterns** — the set of moves permitted at each cell $(i,j)$ in the DP table.

**Type I (Symmetric, no slope constraint):**
Steps: $\{(1,0), (0,1), (1,1)\}$ — standard unconstrained DTW.
$$D(i,j) = C(i,j) + \min(D(i-1,j), D(i,j-1), D(i-1,j-1))$$

**Type II (Sakoe-Chiba symmetric, slope-constrained):**
Steps must satisfy $1/P \leq \Delta j/\Delta i \leq P$ for slope parameter $P$.

**Type III (Asymmetric):**
Only allows moves from $(i-1,j)$ and $(i-1,j-1)$ — the query sequence always advances. Faster but biased.

### Step Weighting

Different step patterns use different weights for the accumulated cost:
- **Symmetric:** $w = (1,1,1)$ for $\{(0,1),(1,0),(1,1)\}$ — uniform penalty per step
- **Weighted symmetric** (Sakoe-Chiba): diagonal step weighted $\times 2$ to normalise path length:
$$D(i,j) = C(i,j) + \min(D(i-1,j) + d_{10}, D(i,j-1) + d_{01}, D(i-1,j-1) + 2 \cdot d_{11})$$

### Normalisation

A key insight: the raw accumulated cost depends on the path length. The **normalised DTW distance** divides by path length:
$$\overline{DTW}(X,Y) = \frac{DTW(X,Y)}{|W^*|}$$

where $|W^*|$ is the number of steps in the optimal path. This makes DTW distances comparable across sequences of different lengths.

### Endpoint Handling

- **Full DTW (global):** Path must start at $(1,1)$ and end at $(n,m)$ — standard gesture recognition
- **Open-end DTW (subsequence):** End point floats — for finding a query within a longer stream (continuous gesture spotting)

---

## Findings Relevant to the Project

| Design choice | Recommendation from Myers & Rabiner |
|---|---|
| Step pattern | Symmetric Type II with slope constraint |
| Path normalisation | Always normalise by path length for comparison across different-length sequences |
| Slope parameter | $P = 2$ (Sakoe-Chiba equivalent) is a good default |
| Endpoint conditions | Global (fixed endpoints) for isolated gesture recognition |

**Practical implication:** When reporting DTW distances, always use the **normalised** form — otherwise, longer sequences will appear more dissimilar simply because the path is longer, not because the shapes differ.

---

## Connection to the $3 Recognizer

The $3 Recognizer circumvents DTW's normalisation issue by resampling all sequences to the same length $N$ before template matching. This is a geometric solution to the same problem that Myers & Rabiner address algorithmically.

---

## Connections

- Foundation for → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW method note]]
- Original DTW → [[Papers/Sakoe1978|Sakoe & Chiba (1978)]]
- Itakura constraint → [[Papers/Itakura1975|Itakura (1975)]]
- Speech context → [[Papers/Rabiner1993_SpeechRecognition|Rabiner & Juang (1993)]] (same author)
- DP foundations → [[Concepts/Dynamic_Programming|Dynamic Programming]]
