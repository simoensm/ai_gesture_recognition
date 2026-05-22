---
title: "Gestures as point clouds: A $P recognizer for user interface prototypes"
authors: ["Vatavu, Radu-Daniel", "Anthony, Lisa", "Wobbrock, Jacob O."]
year: 2012
citekey: "Vatavu2012_P"
doi: "10.1145/2388676.2388732"
venue: "ACM ICMI 2012, pp. 273–280"
type: conference
tags:
  - literature
  - dollar-family
  - point-cloud
  - template-matching
  - multi-stroke
status: read
relevance: low
created: 2026-05-20
---

# Gestures as point clouds: A $P recognizer for user interface prototypes

> [!info] Metadata
> **Authors:** Radu-Daniel Vatavu, Lisa Anthony, Jacob O. Wobbrock
> **Year:** 2012
> **Venue:** ACM ICMI 2012, pp. 273–280
> **DOI:** [10.1145/2388676.2388732](https://doi.org/10.1145/2388676.2388732)
> **Citekey:** `Vatavu2012_P`

---

## Abstract

Extends $1 to multi-stroke gestures by treating each gesture as an unordered point cloud. Achieves rotation and order invariance by matching point clouds optimally via the Hungarian algorithm.

---

## Key Contributions

- Treats gestures as unordered point clouds (no temporal ordering required)
- Optimal point matching via minimum-distance assignment

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/03_Three_Cent_Recognizer|Three Cent Recognizer ($3)]]
- **Role:** `theory` — related family member, not directly used
- **Note:** Our project uses ordered 3D trajectories, so $P's unordered matching is not applicable

## Connections

- Extends → [[Wobbrock2007]]
- See also → [[Kratz2010]], [[Vatavu2012_OneCent]]
