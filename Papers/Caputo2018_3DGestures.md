---
title: "Comparing 3D Trajectories for Simple Mid-Air Gesture Recognition"
authors: ["Caputo, Fabio M.", "Prebianca, Pietro", "Carcangiu, Alessandro", "Spano, Lucio D.", "Giachetti, Andrea"]
year: 2018
citekey: "Caputo2018_3DGestures"
doi: "10.1016/j.cag.2018.02.002"
venue: "Computers & Graphics, 73, 17–25"
type: article
tags: [literature, gesture-recognition, 3d-trajectories, mid-air, template-matching, dtw, comparison, evaluation]
status: read
relevance: high
created: 2026-05-22
---

# Comparing 3D Trajectories for Simple Mid-Air Gesture Recognition

> [!info] Metadata
> **Authors:** Fabio M. Caputo, Pietro Prebianca, Alessandro Carcangiu, Lucio D. Spano, Andrea Giachetti
> **Year:** 2018
> **Venue:** Computers & Graphics, 73, 17–25
> **DOI:** [10.1016/j.cag.2018.02.002](https://doi.org/10.1016/j.cag.2018.02.002)

---

## Abstract

A systematic comparative study of methods for recognising simple **mid-air 3D hand gesture trajectories**. Benchmarks a range of approaches — from classical template-matching (DTW, $1/$3 family) to feature-based and learning-based methods — on multiple 3D gesture datasets. Directly relevant to the project: the same problem setting (3D trajectory, symbol-like gestures, template vs. learning comparison).

---

## Key Contributions

### Methods Compared

| Category | Methods |
|---|---|
| **Template matching** | DTW (with Sakoe-Chiba), $1 Recognizer, $3 Recognizer, Protractor3D |
| **Feature-based** | Hand-crafted geometric descriptors + k-NN / SVM |
| **Learning-based** | SVM on features, shallow neural network |

### Evaluation Protocol
- Multiple datasets with varying numbers of users, classes, and repetitions
- **User-independent** cross-validation (same protocol as the project)
- Reports mean ± std accuracy across folds

### Main Findings

1. **DTW is a strong baseline** — competitive with feature-based methods on small datasets
2. **$3 family** is fast and accurate for well-separated gesture classes; struggles when classes are geometrically similar
3. **Feature-based methods** (velocity, curvature, direction histograms) + SVM can outperform pure template matching when features are carefully engineered
4. **No single winner** — the best method depends on the dataset and the user-independence requirement

---

## Relevance to the Project

This paper is the most directly comparable work to the project:
- Same input: 3D trajectory $r(t) = [x(t), y(t), z(t)]^\top$
- Same task: multiclass classification of symbolic hand gestures
- Same evaluation: user-independent cross-validation
- Tests the same baselines: DTW, $3 Recognizer, edit distance variants

**Use this paper for:**
- Contextualising your results against published benchmarks
- Justifying your choice of preprocessing (resampling, normalisation)
- Citing as motivation for comparing template-matching vs. deep learning

---

## Connections

- DTW comparison → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW]]
- $3 Recognizer → [[Project - Gesture Recognition/03_Three_Cent_Recognizer|$3 Recognizer]]
- Evaluation protocol → [[Concepts/Cross_Validation|Cross-Validation Protocols]]
- Feature engineering → [[Workflows/Data_Preprocessing|Data Preprocessing]]
- SVM alternative → [[Concepts/Kernel_Methods_SVM|Kernel Methods & SVM]]
- Project context → [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition MOC]]
