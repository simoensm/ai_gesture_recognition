# Dynamic Time Warping

Tags: #DTW #signal-alignment #speech-recognition #dynamic-programming

---

## Context & Motivation

In **speech recognition**, the same word can be pronounced at different speeds. Simply comparing two audio signals frame-by-frame would fail.

**Dynamic Time Warping (DTW)** finds the optimal **non-linear alignment** between two time series, allowing for temporal distortions.

---

## Setup

- A **spectrogram** converts audio into a sequence of observation vectors
- **Reference word** $R^n = (r_1^n, \dots, r_I^n)$: $I$ frames
- **Observed word** $O = (o_1, \dots, o_J)$: $J$ frames
- **Distance between frames**: $d(r_i^n, o_j)$ — e.g. Euclidean distance

---

## Constraints (to form a valid [[Directed Acyclic Graph (DAG)]])

| Constraint | Meaning |
|---|---|
| **Monotonicity** | $i_k \geq i_{k-1}$, $j_k \geq j_{k-1}$ — no going backwards |
| **Continuity** | $i_k - i_{k-1} \leq 1$, $j_k - j_{k-1} \leq 1$ — no large jumps |
| **Boundary conditions** | $i_1 = 1$, $j_1 = 1$, $i_K = I$, $j_K = J$ |

These constraints ensure the alignment is **meaningful and complete**.


---

## Dynamic Programming Recurrence

**Initialization:**
$$g(1, 1) = d(r_1^n, o_1)$$

**Recurrence** (for $i = 1,\dots,I$; $j = 1,\dots,J$):
$$g(i, j) = \min \begin{cases} g(i-1, j) + d(r_i^n, o_j) \\ g(i-1, j-1) + 2\,d(r_i^n, o_j) \\ g(i, j-1) + d(r_i^n, o_j) \end{cases}$$

**Final DTW distance:**
$$D(R^n, O) = \frac{1}{I+J}\,g(I, J)$$

(normalized by path length)

---

## Application: Word Recognition

1. For each reference word in the database, compute the DTW distance to the observed word
2. Select the reference word with the **minimum DTW distance** (nearest neighbour)

This is the same idea as [[Edit Distance (Levenshtein)]] but applied to **continuous signals**.

---

## Connection to DP

DTW is a direct application of [[Dynamic Programming]] and [[Bellman Recurrence Relations]] on a 2D alignment grid, which forms a [[Directed Acyclic Graph (DAG)]] thanks to the monotonicity constraints.

---

*Part of → [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
