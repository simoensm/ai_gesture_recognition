---
title: "k-Nearest Neighbour (k-NN) Classifier"
tags:
  - concept
  - classification
  - knn
  - non-parametric
created: 2026-05-20
---

# k-Nearest Neighbour (k-NN) Classifier

> [!abstract] One-line definition
> A non-parametric classifier that assigns the label of the $k$ closest training samples (by some distance measure) to a new query point. No training phase — just store all examples.

---

## Algorithm

Given query $q$ and training set $\mathcal{T} = \{(x_i, y_i)\}$:

1. Compute $d(q, x_i)$ for all $i$ using a chosen distance measure
2. Find the $k$ training samples with smallest distance
3. Predict: majority vote among the $k$ labels (for $k=1$: label of the single nearest neighbour)

$$\hat{y} = \text{majority}\{y_i : x_i \in \mathcal{N}_k(q)\}$$

where $\mathcal{N}_k(q)$ is the set of $k$ nearest neighbours of $q$.

---

## In This Project: 1-NN with Sequence Distances

All three baseline methods use **1-NN** ($k=1$) with different distance measures:

| Method | Distance | Input |
|---|---|---|
| [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping\|DTW]] | DTW distance | Raw 3D sequences |
| [[Project - Gesture Recognition/02_Edit_Distance\|Edit Distance]] | Levenshtein distance | Quantized strings |
| [[Project - Gesture Recognition/03_Three_Cent_Recognizer\|$3 Recognizer]] | Point-cloud distance | Normalized 3D sequences |

For 1-NN, the predicted label is simply:
$$\hat{y} = y_{i^*}, \quad i^* = \arg\min_i\ d(q, x_i)$$

---

## Why 1-NN?

- **No training phase** — all learning is implicit in the stored examples
- **1-NN DTW is a hard-to-beat baseline** on time series (see [[Papers/Ding2008]])
- For gesture recognition, using the **closest known gesture** as the prediction is intuitive

---

## Cross-Validation Interaction

In leave-one-user-out (user-independent) CV:
- Training set = all gestures from 9 users
- Query = gestures from the held-out user
- 1-NN finds the closest training gesture

In leave-one-sample-out (user-dependent) CV:
- Training set = 9 repetitions of each gesture per user
- Query = the held-out repetition
- 1-NN finds the closest training repetition from the same user

See: [[Concepts/Cross_Validation|Cross-Validation Protocol]]

---

## Connections

- Used in → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW]], [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance]], [[Project - Gesture Recognition/03_Three_Cent_Recognizer|$3]]
- Justified by → [[Papers/Ding2008]]
- Concept origin → [[Papers/Sakoe1978]]
