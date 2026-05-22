---
title: "Protractor3D: A closed-form solution to rotation-invariant 3D gestures"
authors: ["Kratz, Sven", "Rohs, Michael"]
year: 2011
citekey: "Kratz2011_Protractor3D"
doi: "10.1145/1943403.1943468"
venue: "ACM IUI 2011, pp. 371–374"
type: conference
tags:
  - literature
  - dollar-family
  - three-cent
  - rotation-invariance
  - kabsch-algorithm
  - svd
  - 3d
  - sota
status: read
relevance: high
created: 2026-05-20
---

# Protractor3D: A closed-form solution to rotation-invariant 3D gestures

> [!info] Metadata
> **Authors:** Sven Kratz, Michael Rohs
> **Year:** 2011
> **Venue:** ACM International Conference on Intelligent User Interfaces (IUI), pp. 371–374
> **DOI:** [10.1145/1943403.1943468](https://doi.org/10.1145/1943403.1943468)
> **Citekey:** `Kratz2011_Protractor3D`

---

## Abstract

A direct follow-up to [[Kratz2010]] that replaces the iterative Golden Section Search with an analytically optimal rotation computed via SVD (the Kabsch algorithm). Protractor3D finds the globally optimal 3D rotation in closed form, eliminating the iterative search entirely. This is both faster and more accurate than GSS.

---

## Key Contributions

- **Closed-form optimal 3D rotation** using SVD (the Kabsch / Wahba problem):
$$R^* = \arg\min_{R \in SO(3)} \sum_{i=1}^N \|R\, g_i - t_i\|^2$$
  Solution: $H = G^\top T$, $H = U\Sigma V^\top$, $R^* = V U^\top$ (with det correction)
- Eliminates iterative GSS — one SVD call per template comparison
- Proved to be faster and at least as accurate as $3 with GSS

## Core Methods & Algorithms

**Kabsch algorithm:**
1. Compute cross-covariance $H = G^\top T$ where $G, T$ are $(N, D)$ point matrices
2. SVD: $H = U \Sigma V^\top$
3. Correction: $d = \det(VU^\top)$, diagonal $D = \text{diag}(1, \ldots, 1, d)$
4. $R^* = VDU^\top$ — ensures proper rotation ($\det = +1$)

## Results & Findings

- Outperforms $3 (GSS) on accuracy
- Significantly faster — no iterative search

## Critical Analysis

**Strengths:**
- Globally optimal rotation — no local minima
- Elegant: single SVD call
- Directly applicable to any dimensionality D (multi-joint skeleton)

**Limitations:**
- In very high dimensions ($D = 63$ for multi-joint), the SVD is on a $63 \times 63$ matrix — computationally heavier than the 3D case, but still fast in practice

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer ($3)]]
- **Role:** `sota` — provides the `kabsch_rotation()` function used as our default
- **Key insight I'm using:** `kabsch_rotation(P, Q)` in `03_Three_Cent_Recognizer.md` implements this exactly. This is why we default to `rotation_method="kabsch"` over `"gss"`.

## Connections

- Improves → [[Kratz2010]]
- Concept note: [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]]
