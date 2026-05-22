---
title: "A $3 gesture recognizer: Simple gesture recognition for devices equipped with 3D acceleration sensors"
authors: ["Kratz, Sven", "Rohs, Michael"]
year: 2010
citekey: "Kratz2010"
doi: "10.1145/1719970.1720051"
venue: "ACM IUI 2010, pp. 341–344"
type: conference
tags:
  - literature
  - dollar-family
  - three-cent
  - 3d
  - template-matching
  - accelerometer
  - sota
status: read
relevance: high
created: 2026-05-20
---

# A $3 gesture recognizer: Simple gesture recognition for devices equipped with 3D acceleration sensors

> [!info] Metadata
> **Authors:** Sven Kratz, Michael Rohs
> **Year:** 2010
> **Venue:** ACM International Conference on Intelligent User Interfaces (IUI), pp. 341–344
> **DOI:** [10.1145/1719970.1720051](https://doi.org/10.1145/1719970.1720051)
> **Citekey:** `Kratz2010`

---

## Abstract

Extends the $1 Recognizer from 2D strokes to 3D trajectory data from accelerometer sensors. The key adaptations: 3D arc-length resampling, PCA-based principal axis alignment, 1D Golden Section Search for rotation around the dominant axis, and scaling to a unit bounding cube. Achieves recognition accuracy competitive with more complex methods.

---

## Key Contributions

- First extension of the $1 paradigm to 3D trajectory data
- 3D arc-length resampling preserving spatial trajectory structure
- Two-stage rotation normalization: PCA alignment + 1D GSS
- Uniform cube scaling for scale normalization in $\mathbb{R}^3$

## Core Methods & Algorithms

1. **Resample** to $N$ points via 3D arc-length parameterization
2. **Translate** centroid to origin
3. **PCA alignment**: rotate so first PC aligns with $x$-axis
4. **Scale** bounding box to $[-1,1]^3$ cube
5. **GSS rotation** (1D search around dominant axis) + point-cloud distance

## Results & Findings

- Competitive accuracy on accelerometer gesture data
- Simple enough to run in real-time on mobile hardware

## Critical Analysis

**Strengths:**
- Direct, interpretable extension of a well-understood 2D algorithm
- No learning phase — just store geometric templates
- Fast at inference (pure geometric operations)

**Limitations:**
- GSS is iterative (10+ function evaluations per template comparison); Kabsch SVD is faster and more accurate → [[Kratz2011_Protractor3D]]
- Designed for single-sensor accelerometers; multi-joint skeleton requires adaptation
- Shape-only: ignores gesture speed/dynamics

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer ($3)]]
- **Role:** `sota` — the primary method we implement
- **Key insight I'm using:** Our `ThreeCentRecognizer` class implements this pipeline. We use Kabsch SVD (`rotation_method="kabsch"`) instead of GSS by default, following [[Kratz2011_Protractor3D]].

## Connections

- Extends → [[Wobbrock2007]]
- Improved rotation → [[Kratz2011_Protractor3D]]
- See also → [[Li2010_Protractor]], [[Vatavu2012_P]]
