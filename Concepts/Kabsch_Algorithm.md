---
title: "Kabsch Algorithm (Optimal Rotation via SVD)"
tags:
  - concept
  - kabsch-algorithm
  - svd
  - rotation
  - linear-algebra
created: 2026-05-20
---

# Kabsch Algorithm (Optimal Rotation via SVD)

> [!abstract] One-line definition
> The Kabsch algorithm finds the optimal rotation matrix $R^* \in SO(D)$ that minimises the RMSD between two sets of corresponding points $P$ and $Q$, computed in closed form via SVD.

---

## Problem Statement

Given two $(N \times D)$ point matrices $P$ and $Q$ (both centred at the origin), find:

$$R^* = \arg\min_{R \in SO(D)} \sum_{i=1}^N \|R\, p_i - q_i\|^2$$

---

## Algorithm

1. Compute the **cross-covariance matrix**: $H = P^\top Q \in \mathbb{R}^{D \times D}$
2. **SVD decomposition**: $H = U \Sigma V^\top$
3. **Handle reflections** (ensure $\det(R^*) = +1$):
   $$d = \det(V U^\top), \quad D_{\text{corr}} = \text{diag}(1, \ldots, 1, d)$$
4. **Optimal rotation**: $R^* = V D_{\text{corr}} U^\top$

The reflection correction (step 3) is necessary because SVD can return a reflection ($\det = -1$) for degenerate cases.

---

## Properties

- **Globally optimal:** Guaranteed to find the rotation that minimises RMSD (no local optima)
- **$\mathcal{O}(ND + D^3)$:** Fast — a single SVD of a small $D \times D$ matrix
- **General dimensionality:** Works for any $D$, including the high-dimensional case ($D=63$ for multi-joint flattened skeleton)

---

## Why Better Than Golden Section Search (GSS)

| | Kabsch (SVD) | GSS |
|---|---|---|
| Optimality | **Global** | Local (1D search only) |
| Speed | $\mathcal{O}(D^3)$ once | $\mathcal{O}(n_{\text{iters}} \cdot ND)$ |
| Applicable to 3D+ | **Yes** | Only 1D rotation angle |

---

## Used In

- [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer]] — `kabsch_rotation(P, Q)` function
- [[Papers/Kratz2011_Protractor3D]] — original Protractor3D paper

## Connections

- Also known as: Wahba's problem (aerospace engineering), Procrustes analysis (statistics)
- Used in → [[Project - Gesture Recognition/03_Three_Cent_Recognizer|03_Three_Cent_Recognizer]]
- Introduced for gesture recognition in → [[Papers/Kratz2011_Protractor3D]]
