---
tags:
  - dynamic-programming
  - recurrence
  - edit-distance
  - lcs
  - question
---
 >Write and explain (without proof) the dynamic programming recurrence formula and apply it to the problem of comparing two sequences of symbols (edit-distance + longest common subsequence), with the resulting equations. As an exercise, solve a concrete example comparing two short sequences (provided at the exam).

---

## Answer

### Dynamic Programming Recurrence (without proof)

Given a DAG with levels and states, the optimal cost-to-go satisfies the **Bellman recurrence**:

$$D^*(s_k) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + D^*(s_{k+1}) \right\}$$

with $D^*(s_N) = 0$. This is computed backwards from level $N$ to $0$. The key insight is that the optimal tail of a path is itself optimal (**principle of optimality**), allowing the minimisation to be split level by level.

---

### Application 1: Edit Distance (Levenshtein)

**Goal:** find the minimum number of editing operations to transform string $\mathbf{x} = x_1\dots x_{|\mathbf{x}|}$ into $\mathbf{y} = y_1\dots y_{|\mathbf{y}|}$.

**Three operations:**

| Operation | Transition | Cost |
|---|---|---|
| Insertion | $(i,\ j-1) \to (i,\ j)$ | $1$ |
| Deletion | $(i-1,\ j) \to (i,\ j)$ | $1$ |
| Substitution | $(i-1,\ j-1) \to (i,\ j)$ | $0$ if $x_i=y_j$, else $1$ |

**State representation:** $(i,j)$ = $i$ characters read from $\mathbf{x}$, $j$ characters written to $\mathbf{y}$. Level $k = i+j$.

**Recurrence:**

$$D^*(i,j) = \min \begin{cases} D^*(i,j-1)+1 & \text{(insertion)} \\ D^*(i-1,j)+1 & \text{(deletion)} \\ D^*(i-1,j-1)+\delta_{ij} & \text{(substitution, } \delta_{ij}=0 \text{ if } x_i=y_j \text{)} \end{cases}$$

**Initialisation:** $D^*(0,j) = j$, $D^*(i,0) = i$.

**Result:** $\text{dist}(\mathbf{x},\mathbf{y}) = D^*(|\mathbf{x}|, |\mathbf{y}|)$.

---

### Application 2: Longest Common Subsequence (LCS)

When the substitution penalty = 2 (double the insertion/deletion cost), edit distance and LCS are related by:

$$\text{dist}(\mathbf{x},\mathbf{y}) = |\mathbf{x}| + |\mathbf{y}| - 2\,\text{lcs}(\mathbf{x},\mathbf{y})$$

Rearranged:

$$\text{lcs}(\mathbf{x},\mathbf{y}) = \frac{1}{2}\bigl(|\mathbf{x}| + |\mathbf{y}| - \text{dist}(\mathbf{x},\mathbf{y})\bigr)$$

This holds because with substitution cost = 2, it is never cheaper to substitute than to delete + insert, so the edit path only uses insertions and deletions — which correspond exactly to non-common characters.

---

### Concrete Example: "lire" vs "livre"

$\mathbf{x}$ = `lire` (length 4), $\mathbf{y}$ = `livre` (length 5).

**DP table** — cell $(i,j)$ = $D^*(i,j)$:

|  | $\varnothing$ | **l** | **i** | **v** | **r** | **e** |
|---|---|---|---|---|---|---|
| $\varnothing$ | 0 | 1 | 2 | 3 | 4 | 5 |
| **l** | 1 | 0 | 1 | 2 | 3 | 4 |
| **i** | 2 | 1 | 0 | 1 | 2 | 3 |
| **r** | 3 | 2 | 1 | 1 | 1 | 2 |
| **e** | 4 | 3 | 2 | 2 | 2 | **1** |

$$\text{dist}(\text{lire}, \text{livre}) = 1$$

**Backtracking:** match **e**, match **r**, **delete v**, match **i**, match **l** → one deletion.

**LCS:**

$$\text{lcs}(\text{lire}, \text{livre}) = \frac{1}{2}(4+5-1) = 4$$

The LCS is `lire` (all 4 characters of `lire` appear in order in `livre`).
