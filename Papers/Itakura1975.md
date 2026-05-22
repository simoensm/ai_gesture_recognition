---
title: "Minimum Prediction Residual Principle Applied to Speech Recognition"
authors: ["Itakura, Fumitada"]
year: 1975
citekey: "Itakura1975"
doi: "10.1109/TASSP.1975.1162641"
venue: "IEEE Transactions on Acoustics, Speech, and Signal Processing, 23(1), 67–72"
type: article
tags: [literature, dtw, dynamic-time-warping, speech-recognition, path-constraint, itakura-parallelogram, sequence-alignment]
status: read
relevance: high
created: 2026-05-22
---

# Minimum Prediction Residual Principle Applied to Speech Recognition

> [!info] Metadata
> **Authors:** Fumitada Itakura (NTT Electrical Communications Laboratory, Japan)
> **Year:** 1975
> **Venue:** IEEE Transactions on Acoustics, Speech, and Signal Processing, 23(1), 67–72
> **DOI:** [10.1109/TASSP.1975.1162641](https://doi.org/10.1109/TASSP.1975.1162641)

---

## Abstract

One of the earliest papers applying DTW-style dynamic programming to speech recognition. Introduces both LPC (Linear Predictive Coding) features for speech and the **Itakura parallelogram** — a slope constraint on the DTW warping path that prevents degenerate alignments by bounding the maximum slope of the path.

---

## Key Contributions

### LPC Distance Measure
Proposes the **Itakura-Saito distance** between LPC spectral envelopes as the local cost $\delta(x_i, y_j)$ in DTW:
$$d_{IS}(x, y) = \frac{x^\top A_y x}{x^\top A_x x} - \log\frac{x^\top A_y x}{x^\top A_x x} - 1 \geq 0$$
where $A_x$, $A_y$ are the LPC correlation matrices of frames $x$ and $y$.

This was the standard local distance for speech recognition before MFCCs.

### Itakura Parallelogram (Slope Constraint)

The key structural contribution for DTW. Constrains the warping path $W = (w_1, \ldots, w_K)$ such that the **local slope** is bounded:
$$\frac{1}{2} \leq \frac{\Delta j}{\Delta i} \leq 2$$

This means the path can move at most two steps along one axis for every step along the other. The accessible region in the $(i,j)$ DP table forms a **parallelogram** shape.

**Formal constraint** — only the following step patterns are allowed:
- $(+1, +1)$ — diagonal
- $(+1, +2)$ — steep in $j$
- $(+2, +1)$ — steep in $i$

Compared to the **Sakoe-Chiba band** (which is a symmetric diagonal band), the Itakura parallelogram enforces a slope constraint rather than a maximum absolute deviation from the diagonal.

### Effect on Computation
The parallelogram reduces the number of accessible cells from $\mathcal{O}(nm)$ to $\mathcal{O}(nm/2)$ in typical cases. More importantly, it **prevents pathological alignments** where a single frame maps to the entire opposing sequence.

---

## Itakura Parallelogram vs. Sakoe-Chiba Band

| Constraint | Type | Shape in DP table | Controls |
|---|---|---|---|
| **Sakoe-Chiba** (1978) | Absolute deviation | Diagonal band $|i-j|\leq r$ | Maximum lag between sequences |
| **Itakura** (1975) | Local slope | Parallelogram | Maximum compression/expansion ratio |

Both serve the same purpose — preventing degenerate alignments — but with different geometric constraints. The Sakoe-Chiba band is more commonly used in modern gesture recognition because its radius $r$ has an intuitive interpretation in terms of temporal lag.

---

## Relevance to Gesture Recognition

In the project:
- The DTW baseline uses a **Sakoe-Chiba band** by default (`window=0.1`)
- The Itakura constraint is an alternative ablation: replace `global_constraint="sakoe_chiba"` with `global_constraint="itakura"` in tslearn
- For gesture data, the Itakura constraint may be too strict if gestures have large local speed variations (fast strokes alternating with slow positioning)

---

## Connections

- DTW algorithm → [[Papers/Sakoe1978|Sakoe & Chiba (1978)]]
- Used in → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW method note]]
- Dynamic programming backbone → [[Concepts/Dynamic_Programming|Dynamic Programming]]
- Speech recognition context → [[Papers/Rabiner1993_SpeechRecognition|Rabiner & Juang (1993)]]
