---
title: "Vector Quantization (VQ) / Codebook"
tags:
  - concept
  - vector-quantization
  - clustering
  - k-means
  - sequence-encoding
created: 2026-05-20
---

# Vector Quantization (VQ) / Codebook

> [!abstract] One-line definition
> Mapping continuous feature vectors to a finite set of discrete symbols (a "codebook") using nearest-centroid assignment, typically via k-means clustering.

---

## Why It's Needed

Edit distance and LCS operate on **symbolic strings**, not continuous vectors. To apply these methods to 3D gesture data, each frame $x_t \in \mathbb{R}^D$ must be mapped to a discrete symbol from a finite alphabet $\Sigma = \{1, \ldots, K\}$.

This mapping is called **vector quantization** (or **codebook encoding**).

---

## Algorithm

**Training (fit codebook):**
1. Pool all frames from all training gestures: $\{x_t\}_{t,i} \in \mathbb{R}^D$
2. Run k-means with $K$ clusters → centroids $\{c_1, \ldots, c_K\} \subset \mathbb{R}^D$

**Encoding (quantize):**
$$\text{symbol}(x_t) = \arg\min_{k \in \{1,\ldots,K\}} \|x_t - c_k\|_2$$

Each gesture becomes a string: $S = s_1 s_2 \cdots s_T$ with $s_i \in \{1, \ldots, K\}$.

---

## Key Parameter: Codebook Size $K$

| $K$ | Effect |
|---|---|
| Small (8) | Coarse: many frames share a symbol; robust to noise but low discriminability |
| Medium (16) | Good balance — **default in ablation** |
| Large (64) | Fine: rich alphabet, more sensitive to noise |

Larger $K$ → more distinct symbols → richer string representation, but noisier edit distances.

---

## Codebook Leakage Warning

> [!warning] Important for cross-validation
> **The codebook must be fitted only on the training fold.** If you fit the codebook on the full dataset (including test), you introduce **data leakage** — the test set's frames influence the codebook, giving overly optimistic results.
>
> The guidelines acknowledge this and allow fitting the codebook once on the full dataset for simplicity, but note that the rigorous approach fits it inside each cross-validation fold.

---

## Used In

- [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance baseline]] — quantizes gestures before Levenshtein DP
- Potentially in LCS baseline (third optional baseline)

## Connections

- Applied by → [[Papers/Nyirarugira2014]]
- Implemented as `GestureCodebook` class in [[Project - Gesture Recognition/02_Edit_Distance|02_Edit_Distance]]
- Related concept: [[kNN_Classifier|k-NN Classifier]]
