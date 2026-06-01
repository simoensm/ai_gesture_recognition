# Editing Operations

Tags: #edit-distance #string-operations

---

## Context

Used in [[Edit Distance (Levenshtein)]] to transform string **x** into string **y**.

---

## The Three Operations

### 1. Insertion
Add a character to **y** without consuming a character from **x**.

- **Transition:** $(i,\ j-1) \to (i,\ j)$
- **Cost:** $\delta_\text{ins}(y_j) = 1$
- **Level jump:** $k \to k+1$

### 2. Deletion
Consume a character from **x** without adding it to **y**.

- **Transition:** $(i-1,\ j) \to (i,\ j)$
- **Cost:** $\delta_\text{del}(x_i) = 1$
- **Level jump:** $k \to k+1$

### 3. Substitution
Consume a character from **x** and write a character to **y** (may be identical = free).

- **Transition:** $(i-1,\ j-1) \to (i,\ j)$
- **Cost:**

$$\delta_{ij} = \begin{cases} 0 & \text{if } x_i = y_j \\ 1 & \text{if } x_i \neq y_j \end{cases}$$

- **Level jump:** $k \to k+2$ (skips one level → needs dummy node in [[Directed Acyclic Graph (DAG)]])

---

## Summary Table

| Operation | Reads from x? | Writes to y? | Cost |
|---|---|---|---|
| Insertion | No | Yes | 1 |
| Deletion | Yes | No | 1 |
| Substitution | Yes | Yes | 0 or 1 |

---

*Part of → [[Edit Distance (Levenshtein)]] | [[MOC - Modelling Sequential Data]]*
