---
title: "Protractor: A fast and accurate gesture recognizer"
authors: ["Li, Yang"]
year: 2010
citekey: "Li2010_Protractor"
doi: "10.1145/1753326.1753654"
venue: "ACM CHI 2010, pp. 2169–2172"
type: conference
tags:
  - literature
  - dollar-family
  - rotation-invariance
  - golden-section-search
  - 2d
status: read
relevance: low
created: 2026-05-20
---

# Protractor: A fast and accurate gesture recognizer

> [!info] Metadata
> **Authors:** Yang Li
> **Year:** 2010
> **Venue:** ACM CHI 2010, pp. 2169–2172
> **DOI:** [10.1145/1753326.1753654](https://doi.org/10.1145/1753326.1753654)
> **Citekey:** `Li2010_Protractor`

---

## Abstract

Proposes a closed-form optimal angle for 2D gesture rotation alignment using the inner product between point sequences, replacing the iterative GSS of $1. Fast and effective for 2D.

---

## Key Contributions

- Closed-form optimal rotation angle for 2D gesture matching (golden section angle via dot product)
- Significantly faster than GSS while maintaining accuracy

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer ($3)]]
- **Role:** `theory` — historical context for rotation-optimal matching; the 3D version is [[Kratz2011_Protractor3D]]

## Connections

- Extends → [[Wobbrock2007]]
- 3D version → [[Kratz2011_Protractor3D]]
