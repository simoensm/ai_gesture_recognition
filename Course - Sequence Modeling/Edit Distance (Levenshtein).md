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

This maps the problem onto a [[Directed Acyclic Graph (DAG)]] and [[Dynamic Programming]] can be applied.

---

## The Three Editing Operations

See → [[Editing Operations]]

| Operation | Transition | Cost |
|---|---|---|
| Insertion | $(\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^{j-1}) \to (\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^j)$ | $\delta_\text{ins}(y_i) = 1$ |
| Deletion | $(\mathbf{x}_{i-1}^{|\mathbf{x}|}, \mathbf{y}_0^j) \to (\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^j)$ | $\delta_\text{del}(x_i) = 1$ |
| Substitution | $(\mathbf{x}_{i-1}^{|\mathbf{x}|}, \mathbf{y}_0^{j-1}) \to (\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^j)$ | $\delta_{ij} = 0$ if $x_i = y_j$, else $1$ |

---

## Dynamic Programming Formula

**Initialization:**
$$D^*(\mathbf{x}_0^{|\mathbf{x}|}, \mathbf{y}_0^0) = 0$$

**Recurrence:**
$$D^*(\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^j) = \min \begin{cases} D^*(\mathbf{x}_i^{|\mathbf{x}|}, \mathbf{y}_0^{j-1}) + 1 & \text{(insertion)} \\ D^*(\mathbf{x}_{i-1}^{|\mathbf{x}|}, \mathbf{y}_0^j) + 1 & \text{(deletion)} \\ D^*(\mathbf{x}_{i-1}^{|\mathbf{x}|}, \mathbf{y}_0^{j-1}) + \delta_{ij} & \text{(substitution)} \end{cases}$$

Each level $(i+j) = \text{const}$ corresponds to a **diagonal of the 2D table**.

---

## Example: "lire" vs "livre"

$$\text{dist}(\text{lire}, \text{livre}) = 1$$

(one deletion of 'v')

---

## Relation to Longest Common Subsequence

See → [[Longest Common Subsequence]]

$$\text{lcs}(\mathbf{x}, \mathbf{y}) = \frac{1}{2}(|\mathbf{x}| + |\mathbf{y}| - \text{dist}(\mathbf{x}, \mathbf{y}))$$

Valid when the substitution penalty = 2 × insertion/deletion cost.

---

## Finding the Optimal Path

To recover the **optimal sequence of operations**:
1. Maintain a pointer to the predecessor state in each cell
2. **Backtrack** from the final state $(|\mathbf{x}|, |\mathbf{y}|)$

---

*Part of → [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
