---
title: "Gesture-based system for next generation natural and intuitive interfaces"
authors: ["Huang", "Jaiswal", "Rai"]
year: 2019
citekey: "Huang2019"
doi: ""
venue: "Artificial Intelligence for Engineering Design, Analysis and Manufacturing, 33(1), 54–68"
type: article
tags:
  - literature
  - dataset
  - gesture-recognition
  - 3d
  - benchmark
status: read
relevance: critical
created: 2026-05-20
---

# Gesture-based system for next generation natural and intuitive interfaces

> [!info] Metadata
> **Authors:** Huang, Jaiswal, Rai
> **Year:** 2019
> **Venue:** Artificial Intelligence for Engineering Design, Analysis and Manufacturing, 33(1), 54–68
> **Citekey:** `Huang2019`

---

## Abstract

Presents a gesture recognition system for engineering design interfaces and, critically, introduces the **dataset** used in our project. The dataset contains 3D hand tracking data from 10 users, each performing 10 repetitions of 10 gesture classes (digits 0–9 in domain 1, 3D figures in domain 4), giving 1000 sequences per domain.

---

## Key Contributions

- **Primary source of the project dataset**: 3D position vector $\mathbf{r}(t) = [x(t), y(t), z(t)]^T$ for hand gestures
- Dataset structure:
  - **Domain 1:** 10 users × 10 digits (0–9) × 10 repetitions = **1,000 sequences**
  - **Domain 4:** 10 users × 10 3D figures × 10 repetitions = **1,000 sequences**
- Variable-length sequences (different users → different speeds)
- Provides benchmark for gesture recognition systems

## Dataset Details (Critical for Project)

| Property | Value |
|---|---|
| Users | 10 subjects |
| Gesture classes | 10 per domain |
| Repetitions | 10 per user per class |
| Total sequences | 1,000 per domain |
| Features | 3D coordinates: x(t), y(t), z(t) |
| Time steps | Variable (different speeds) |
| Domain 1 | Hand-drawn digits 0–9 |
| Domain 4 | 3D geometric figures |

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Role:** `dataset` — **THE** dataset we use. This is the most critical reference.
- **Key detail:** The paper is available on Moodle (see footnote 2 in guidelines)
- **Cross-validation:** The leave-one-user-out (user-independent) and leave-one-sample-out (user-dependent) protocols are designed for this 10×10×10 structure

## Connections

- Dataset used in → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW]], [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance]], [[Project - Gesture Recognition/03_Three_Cent_Recognizer|$3]], [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]]
- Concept note → [[Concepts/Cross_Validation|Cross-Validation Protocol]]

---

## My Notes

**Domain 1** = digits 0–9 = 10 classes. This is our primary benchmark.
**Domain 4** = 3D figures = 10 classes. Must also be analysed in the report.

The dataset is publicly available; check the paper on Moodle for the download link. Sequences have variable length → need resampling before all methods except raw DTW.
