---
title: "Longest Common Subsequence (LCS)"
tags: [concept, lcs, dynamic-programming, edit-distance, sequence-alignment, string-matching, gesture-recognition, baseline]
created: 2026-05-22
---

# Longest Common Subsequence (LCS)

> A fundamental sequence comparison algorithm that finds the longest subsequence present in both sequences in the same relative order — without requiring the symbols to be contiguous. A core baseline option for gesture recognition alongside [[Concepts/Dynamic_Programming|DTW]] and [[Project - Gesture Recognition/02_Edit_Distance|edit distance]].

---

## 1. Definition

Given two sequences $X = (x_1, x_2, \ldots, x_m)$ and $Y = (y_1, y_2, \ldots, y_n)$ over an alphabet $\Sigma$, a **common subsequence** is a sequence $Z$ that is a subsequence of both $X$ and $Y$ — meaning $Z$ can be obtained from each by deleting some elements (without reordering).

The **LCS** is the longest such $Z$.

**Example:**
```
X = A B C B D A B
Y = B D C A B A
LCS = B C B A   (length 4)
```

Note: $Z$ does not need to be contiguous in either $X$ or $Y$ — only the relative order must be preserved.

---

## 2. Dynamic Programming Solution

### Recurrence

Define $L(i,j)$ = length of the LCS of $X_{1:i}$ and $Y_{1:j}$:

$$L(i,j) = \begin{cases}
0 & \text{if } i=0 \text{ or } j=0 \\
L(i-1,j-1) + 1 & \text{if } x_i = y_j \\
\max(L(i-1,j),\; L(i,j-1)) & \text{if } x_i \neq y_j
\end{cases}$$

**Time complexity:** $\mathcal{O}(mn)$ — identical to edit distance.  
**Space complexity:** $\mathcal{O}(mn)$ for full table; $\mathcal{O}(\min(m,n))$ with rolling arrays.

### DP Table Example

```
    ""  B  D  C  A  B  A
""   0  0  0  0  0  0  0
A    0  0  0  0  1  1  1
B    0  1  1  1  1  2  2
C    0  1  1  2  2  2  2
B    0  1  1  2  2  3  3
D    0  1  2  2  2  3  3
A    0  1  2  2  3  3  4
B    0  1  2  2  3  4  4
```

LCS length = $L(7,6) = 4$.

### Backtracking
To recover the actual LCS (not just its length), backtrack from $L(m,n)$:
- If $x_i = y_j$: take this symbol, move to $L(i-1,j-1)$
- Else: move to whichever of $L(i-1,j)$ or $L(i,j-1)$ is larger

---

## 3. LCS as a Similarity Measure

The LCS-based dissimilarity between two sequences of length $m$ and $n$:
$$d_{LCS}(X,Y) = 1 - \frac{2 \cdot |LCS(X,Y)|}{|X| + |Y|}$$

This is the **normalised LCS distance** ∈ [0,1]:
- 0 → identical sequences
- 1 → no common subsequence

For gesture classification, use as the distance in a [[Concepts/kNN_Classifier|1-NN classifier]].

---

## 4. LCS vs. Edit Distance

Both are computed by $\mathcal{O}(mn)$ DP on quantised symbol sequences, but they measure different things:

| | LCS | Edit Distance |
|---|---|---|
| **What it measures** | Length of longest shared subsequence | Minimum operations to transform one into the other |
| **Operations** | Matches only (no substitution) | Insert, delete, **substitute** |
| **Similarity vs. distance** | Similarity (maximise) | Distance (minimise) |
| **Sensitivity** | Insensitive to insertions/deletions between matches | Penalises every misalignment |
| **Relation** | $d_{edit}(X,Y) = m + n - 2|LCS(X,Y)|$ when substitution cost = 2 |  |

**Key relationship:** when substitution cost equals insert+delete cost (= 2), edit distance and LCS are equivalent. In practice, LCS ignores the *cost* of the gap between matches, making it more tolerant of insertions/deletions.

---

## 5. Application to Gesture Recognition

### Pipeline (identical to edit distance)

```
3D gesture sequence
        │
        ▼
Vector Quantization (k-means codebook)
  Each frame → nearest centroid label
        │
        ▼
Symbolic sequence  e.g. "AAABCCBBA"
        │
        ▼
Compute LCS with all training sequences
        │
        ▼
1-NN: assign label of most similar training sequence
```

### Codebook
Shared with the edit distance baseline — both use the same k-means codebook. Only the similarity computation differs.
→ [[Concepts/Vector_Quantization|Vector Quantization]]

### Hyperparameters
- **Codebook size $K$:** same as edit distance — typically 8 to 64 symbols
- **Sequence length $L$:** resampling length before quantisation
- **Distance normalisation:** raw LCS length vs. normalised distance above

---

## 6. Variants and Extensions

### LCSS (Longest Common SubSequence with Slack)
Agrawal & Srikant (1995) — allows a threshold $\varepsilon$ on the match condition:
$$x_i \approx y_j \quad \text{iff} \quad \|x_i - y_j\| \leq \varepsilon$$

Can be applied directly to **continuous** (non-quantised) sequences — avoids the need for a codebook.

### ERP (Edit Distance with Real Penalty)
Chen & Ng (2004) — hybrid: uses a real-valued gap penalty $g$ (a fixed reference point), giving a proper metric unlike pure edit distance.

### TWE (Time Warp Edit Distance)
Marteau (2009) — combines DTW's elastic alignment with edit distance's insertion/deletion model. Metric property guaranteed.

---

## 7. Python Implementation (Scratch)

```python
import numpy as np

def lcs_length(s: list, t: list) -> int:
    """Compute LCS length via DP. O(mn) time and space."""
    m, n = len(s), len(t)
    # Use rolling arrays for space efficiency
    prev = [0] * (n + 1)
    for i in range(1, m + 1):
        curr = [0] * (n + 1)
        for j in range(1, n + 1):
            if s[i-1] == t[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        prev = curr
    return prev[n]

def lcs_distance(s: list, t: list) -> float:
    """Normalised LCS distance in [0, 1]."""
    lcs = lcs_length(s, t)
    return 1.0 - 2.0 * lcs / (len(s) + len(t))
```

---

## Connections

- Same DP structure as → [[Concepts/Dynamic_Programming|Dynamic Programming]]
- Closely related to → [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance]]
- Requires → [[Concepts/Vector_Quantization|Vector Quantization]] for continuous signals
- Classification → [[Concepts/kNN_Classifier|k-NN Classifier]]
- Referenced in assignment → [[Papers/Theodoridis2009|Theodoridis & Koutroumbas (2009)]]
- Project application → [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance method note]] (same pipeline)
- Alternative for continuous data → DTW → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW]]
