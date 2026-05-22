---
title: "Gestures without libraries, toolkits or training: A $1 recognizer for user interface prototypes"
authors: ["Wobbrock, Jacob O.", "Wilson, Andrew D.", "Li, Yang"]
year: 2007
citekey: "Wobbrock2007"
doi: "10.1145/1294211.1294238"
venue: "ACM UIST 2007, pp. 159–168"
type: conference
tags:
  - literature
  - dollar-family
  - template-matching
  - gesture-recognition
  - 2d
  - sota
status: read
relevance: high
created: 2026-05-20
---

# Gestures without libraries, toolkits or training: A $1 recognizer for user interface prototypes

> [!info] Metadata
> **Authors:** Jacob O. Wobbrock, Andrew D. Wilson, Yang Li
> **Year:** 2007
> **Venue:** ACM Symposium on User Interface Software and Technology (UIST), pp. 159–168
> **DOI:** [10.1145/1294211.1294238](https://doi.org/10.1145/1294211.1294238)
> **Citekey:** `Wobbrock2007`

---

## Abstract

Introduces the $1 Recognizer — a remarkably simple 2D gesture recognizer that achieves competitive accuracy with fewer than 100 lines of code. The key insight is that after a small set of geometric normalizations (resample → rotate → scale → translate), gestures become comparable via point-by-point Euclidean distance with a Golden Section Search for optimal rotation.

---

## Key Contributions

- Introduced the $1 paradigm: resample → rotate → scale → translate → nearest template
- Golden Section Search (GSS) for rotation-invariant matching in $[-45°, 45°]$
- Demonstrated that a 100-line recognizer could match complex library-based systems
- Open-sourced → democratized gesture recognition for HCI prototyping

## Core Methods & Algorithms

1. **Resample** stroke to $N=64$ equidistant points
2. **Indicative angle**: rotate so centroid→first point aligns with 0°
3. **Scale** to $[-1,1]^2$ bounding box
4. **Translate** centroid to origin
5. **Classify**: for each template, find optimal rotation via GSS, compute path distance → 1-NN

## Results & Findings

- Achieves 98.6% accuracy at 9 training samples per class on a 16-class 2D gesture set
- With only 1 training sample: 79.6%

## Critical Analysis

**Strengths:**
- Elegantly simple; easy to implement and understand
- Provides a strong intuition for the geometric normalization pipeline used in $3

**Limitations:**
- Designed for 2D strokes; 3D extension requires the $3 Recognizer
- Rotation via GSS is slow compared to Kabsch SVD (see [[Kratz2011_Protractor3D]])
- Ignores dynamics/velocity (shape-only matching)

---

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer ($3)]]
- **Role:** `sota` — foundation of the 3D extension we use
- **Key insight I'm using:** The geometric normalization pipeline (resample → translate → PCA-align → scale → match) directly extends this to 3D. The GSS is replaced by Kabsch SVD in our implementation.

## Connections

- Extended to 3D by → [[Kratz2010]], [[Kratz2011_Protractor3D]]
- Point-cloud variant → [[Vatavu2012_P]]
- One-cent variant → [[Vatavu2012_OneCent]]
- Faster rotation → [[Li2010_Protractor]]
- Concept note: [[Concepts/kNN_Classifier|k-NN Classifier]]
