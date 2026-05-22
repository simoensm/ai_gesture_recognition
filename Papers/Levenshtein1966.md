---
title: "Binary codes capable of correcting deletions, insertions, and reversals"
authors: ["Levenshtein, Vladimir I."]
year: 1966
citekey: "Levenshtein1966"
doi: ""
venue: "Soviet Physics Doklady, 10(8), 707–710"
type: article
tags:
  - literature
  - edit-distance
  - string-similarity
  - dynamic-programming
  - baseline
status: read
relevance: high
created: 2026-05-20
---

# Binary codes capable of correcting deletions, insertions, and reversals

> [!info] Metadata
> **Authors:** Vladimir I. Levenshtein
> **Year:** 1966
> **Venue:** Soviet Physics Doklady, 10(8), 707–710
> **Citekey:** `Levenshtein1966`

---

## Abstract

The foundational paper defining the **edit distance** (now called Levenshtein distance) between two strings as the minimum number of single-character insertions, deletions, and substitutions to transform one string into the other.

---

## Key Contributions

- Defined the edit distance $\text{lev}(S, T)$ between strings $S$ and $T$
- Proved the metric axioms (non-negativity, identity, symmetry, triangle inequality)
- Original motivation was error-correcting codes, not NLP or gesture recognition

## Core Methods & Algorithms

Recursive definition (as later formalized by Wagner & Fischer):
$$\text{lev}(i,j) = \begin{cases} \max(i,j) & \text{if } \min(i,j) = 0 \\ \min\left(\text{lev}(i-1,j)+1,\ \text{lev}(i,j-1)+1,\ \text{lev}(i-1,j-1)+[s_i \neq t_j]\right) & \text{otherwise} \end{cases}$$

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance baseline]]
- **Role:** `baseline` — foundational algorithm
- **Key insight I'm using:** The uniform-cost Levenshtein distance (ins=del=sub=1) is our baseline edit distance implementation. Weighted variants are explored in the ablation.

## Connections

- Efficient DP implementation → [[Wagner1974]]
- Adapted for gesture recognition → [[Nyirarugira2014]]
- Concept note: [[Concepts/Dynamic_Programming|Dynamic Programming]]
- Concept note: [[Concepts/Vector_Quantization|Vector Quantization]]
