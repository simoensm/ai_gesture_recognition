---
tags:
  - dynamic-programming
  - recurrence
  - dtw
  - dynamic-time-warping
  - speech
  - speech-recognition
---
>Write and explain (without proof) the dynamic programming recurrence formula and apply it to the problem of computing the dynamic time warping distance between two speech signals, with the resulting equations.

---

## Answer

### Dynamic Programming Recurrence (without proof)

Given a DAG with levels and states, the optimal cost-to-go satisfies the **Bellman recurrence**:

$$D^*(s_k) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + D^*(s_{k+1}) \right\}$$

with $D^*(s_N) = 0$. By the **principle of optimality**, the tail of an optimal path is itself optimal, allowing the minimisation to be decomposed level by level.

---

### Motivation: Why DTW?

In speech recognition, the same word can be pronounced at different speeds. A simple frame-by-frame comparison of two audio signals fails when their lengths differ. **Dynamic Time Warping (DTW)** finds the optimal **non-linear alignment** between two time series by allowing one signal to be "stretched" or "compressed" locally to match the other.

---

### Setup

- A **spectrogram** converts raw audio into a sequence of feature vectors (one per frame).
- **Reference word** $R^n = (r_1^n, \dots, r_I^n)$: $I$ frames.
- **Observed word** $O = (o_1, \dots, o_J)$: $J$ frames.
- **Frame distance** $d(r_i^n, o_j)$: e.g. Euclidean distance between feature vectors.

The alignment is a path $(i_1,j_1), \dots, (i_K,j_K)$ through the $I\times J$ grid, subject to:

| Constraint | Condition | Meaning |
|---|---|---|
| Monotonicity | $i_k \geq i_{k-1}$, $j_k \geq j_{k-1}$ | No going backwards |
| Continuity | $i_k - i_{k-1} \leq 1$, $j_k - j_{k-1} \leq 1$ | No large jumps |
| Boundary | $i_1=1$, $j_1=1$, $i_K=I$, $j_K=J$ | Full coverage of both signals |

These constraints make the grid a **DAG**, enabling DP.

---

### DTW Recurrence

**Initialisation:**

$$g(1,1) = d(r_1^n, o_1)$$

**Induction** (for $i=1,\dots,I$; $j=1,\dots,J$):

$$g(i,j) = \min \begin{cases} g(i-1,j) + d(r_i^n, o_j) & \text{(move along reference)} \\ g(i-1,j-1) + 2\,d(r_i^n, o_j) & \text{(diagonal, weighted ×2)} \\ g(i,j-1) + d(r_i^n, o_j) & \text{(move along observation)} \end{cases}$$

**Final DTW distance** (normalised by path length):

$$\boxed{D(R^n, O) = \frac{1}{I+J}\,g(I,J)}$$

The normalisation by $I+J$ makes distances comparable across words of different lengths.

---

### Application: Word Recognition

Given a dictionary of $W$ reference words $\{R^1, R^2, \dots, R^W\}$ and an observed word $O$:

1. Compute $D(R^n, O)$ for each reference word $n$ using the DTW recurrence.
2. Select the reference with **minimum DTW distance**:

$$\hat{n} = \arg\min_n D(R^n, O)$$

This is a nearest-neighbour classifier in DTW distance space.

---

### Connection to Edit Distance

DTW and edit distance are structurally identical DP problems on a 2D grid. The key difference is:
- Edit distance operates on **discrete symbols** with unit costs
- DTW operates on **continuous feature vectors** with real-valued frame distances
- DTW allows diagonal transitions to be weighted differently (×2) to discourage degenerate alignments
