---
tags:
  - edit-distance
  - string-similarity
  - dynamic-programming
  - NLP
  - sequence-modeling
---
Tags: #edit-distance #string-similarity #dynamic-programming #NLP

---

## Definition

The **edit distance** (also called **Levenshtein distance**) measures the **minimum number of [[Editing Operations]]** needed to transform one string **x** into another string **y**.

$$\text{dist}(\mathbf{x}, \mathbf{y}) = D^*\!\left(\mathbf{x}_{|\mathbf{x}|}^{|\mathbf{x}|}, \mathbf{y}_0^{|\mathbf{y}|}\right)$$

---

## Setup

Two character strings:
$$\mathbf{y} = y_1 y_2 \dots y_{|\mathbf{y}|}, \quad \mathbf{x} = x_1 x_2 \dots x_{|\mathbf{x}|}$$

- $|\mathbf{x}|$ = length of string **x** (in general $|\mathbf{x}| \neq |\mathbf{y}|$)
- $\mathbf{x}_i^j = x_i x_{i+1} \dots x_j$ is the substring from $i$ to $j$

---

## State Representation

The process reads **x** character by character to reconstruct **y**.
- **State** = a pair $(i, j)$:
  - $i$ = number of characters read from **x**
  - $j$ = number of characters written to **y**
- **Level** $k$ = all states where $i + j = k = \text{const}$

### Convention on $i$

State $i$ means we have **read (extracted) the first $i$ characters of x**. In other words, $x$ has been cut from its first $i$ characters — they have been consumed in order to construct $y$. The remaining suffix $x_{i+1}\dots x_{|\mathbf{x}|}$ is yet to be processed.

### Convention on $j$

State $j$ means **the first $j$ characters of y have been transcribed**. We progressively read characters from $x$ and write characters to $y$, so $j$ tracks how far along the reconstruction of $y$ we are.

> [!summary] Intuition
> A state $(i, j)$ captures a snapshot of the transformation: "$i$ characters of $x$ have been consumed, and $j$ characters of $y$ have been produced." Each editing operation advances $i$, $j$, or both, moving us forward in the DAG until we reach the final state $(|\mathbf{x}|, |\mathbf{y}|)$.

This maps the problem onto a [[Directed Acyclic Graph (DAG)]] structured into **levels and states**:
- Each **level** $k = i + j$ is a step in the process
- Each **state** is uniquely characterised by the pair $(i, j)$

We can therefore directly apply [[Dynamic Programming]] to find the optimal sequence of operations efficiently.

---

## The Three Editing Operations

See → [[Editing Operations]]

| Operation | Transition | Cost |
|---|---|---|
| Insertion | $(i,\ j-1) \to (i,\ j)$ | $\delta_\text{ins}(y_j) = 1$ |
| Deletion | $(i-1,\ j) \to (i,\ j)$ | $\delta_\text{del}(x_i) = 1$ |
| Substitution | $(i-1,\ j-1) \to (i,\ j)$ | $\delta_{ij} = 0$ if $x_i = y_j$, else $1$ |

---
	
## Dynamic Programming Formula

**Initialization:**
$$D^*(\mathbf{x}_0^{|\mathbf{x}|}, \mathbf{y}_0^0) = 0$$

**Recurrence:**
$$D^*(\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^j) = \min \begin{cases} D^*(\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^{j-1}) + 1 & \text{(insertion)} \\ D^*(\mathbf{x}_{i-1}^{|\mathbf{x}|}, \mathbf{y}_0^j) + 1 & \text{(deletion)} \\ D^*(\mathbf{x}_{i-1}^{|\mathbf{x}|}, \mathbf{y}_0^{j-1}) + \delta_{ij} & \text{(substitution)} \end{cases}$$

Each level $(i+j) = \text{const}$ corresponds to a **diagonal of the 2D table** :
- One **level** corresponds to a diagonal $i + j = \text{const}$
- One **state** corresponds to a cell $(i, j)$
- One **operation** (insertion, deletion, substitution) corresponds to a valid transition between adjacent cells in the table
![[Screenshot 2026-06-01 at 12.41.35.png]]
---

## Example: "lire" vs "livre"

$\mathbf{x}$ = `lire` (length 4), $\mathbf{y}$ = `livre` (length 5). Each cell $(i,j)$ holds $D^*(i,j)$ = minimum edits to transform $x_1\dots x_i$ into $y_1\dots y_j$.

|  | $\varnothing$ | **l** | **i** | **v** | **r** | **e** |
|---|---|---|---|---|---|---|
| $\varnothing$ | 0 | 1 | 2 | 3 | 4 | 5 |
| **l** | 1 | 0 | 1 | 2 | 3 | 4 |
| **i** | 2 | 1 | 0 | 1 | 2 | 3 |
| **r** | 3 | 2 | 1 | 1 | 1 | 2 |
| **e** | 4 | 3 | 2 | 2 | 2 | **1** |

$$\text{dist}(\text{lire}, \text{livre}) = 1$$

The optimal path backtracks from $(4,5)$: match **e**, match **r**, **delete v**, match **i**, match **l** — one deletion of `v`.

---

## Relation to Longest Common Subsequence

See → [[Longest Common Subsequence]]

Starting from the general identity:

$$\text{dist}(\mathbf{x}, \mathbf{y}) = \text{lcs}(\mathbf{x}, \mathbf{x}) + \text{lcs}(\mathbf{y}, \mathbf{y}) - 2\,\text{lcs}(\mathbf{x}, \mathbf{y})$$

Since $\text{lcs}(\mathbf{x}, \mathbf{x}) = |\mathbf{x}|$ and $\text{lcs}(\mathbf{y}, \mathbf{y}) = |\mathbf{y}|$, this simplifies to:

$$\text{dist}(\mathbf{x}, \mathbf{y}) = |\mathbf{x}| + |\mathbf{y}| - 2\,\text{lcs}(\mathbf{x}, \mathbf{y})$$

Which can be rearranged to express LCS in terms of edit distance:

$$\text{lcs}(\mathbf{x}, \mathbf{y}) = \frac{1}{2}\bigl(|\mathbf{x}| + |\mathbf{y}| - \text{dist}(\mathbf{x}, \mathbf{y})\bigr)$$

> [!warning] Condition
> These equalities hold only when the **substitution penalty = 2 × insertion/deletion cost**. Under this condition, a substitution is never preferred over an insertion + deletion pair, so the edit distance only uses insertions and deletions — which is exactly what LCS counts.

**Application to the example:** since $\text{dist}(\text{lire}, \text{livre}) = 1$, we have:

$$\text{lcs}(\text{lire}, \text{livre}) = \frac{1}{2}(4 + 5 - 1) = 4$$

---

## Finding the Optimal Path

To recover the **optimal sequence of operations**:
1. For each cell $(i, j)$ of the DP table, maintain a **pointer to the previous state** (the cell it was reached from).
2. Once the table is filled, **backtrack from the final state** $(|\mathbf{x}|, |\mathbf{y}|)$ by following pointers back to $(0, 0)$.
3. The sequence of pointers traced gives the optimal edit sequence (which operations to apply and in which order).

> [!info] Extensions
> Numerous extensions and generalisations of this basic algorithm have been developed, including:
> - **Weighted edit distance** — different costs for each operation or character pair
> - **Damerau–Levenshtein** — adds transposition as a fourth operation
> - **Restricted edit distance** — constraints on the allowed path (e.g. Sakoe–Chiba band, used in DTW)
> - **Approximate string matching** — find the best match of a pattern within a longer text

---

*Part of → [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
