---
tags:
  - DAG
  - graph
  - dynamic-programming
  - sequence-modeling
---

Tags: #graph-theory #dynamic-programming #structure

---

## Definition

A **Directed Acyclic Graph (DAG)** is a directed graph with **no cycles**.

- Nodes can be organized into **levels**
- Edges only go forward (from lower to higher levels)
- This structure is required to apply [[Dynamic Programming]]

---

## Why DP requires a DAG

[[Dynamic Programming]] relies on the principle that optimal subproblems don't loop back. A cycle would create infinite recursion in the Bellman equations.

The acyclic property guarantees that:
- There is a well-defined topological order
- Each subproblem is solved exactly once

---

## Standard Lattice vs. Shortcut DAG

**Standard lattice:** every transition goes from level $k$ exactly to level $k+1$.

**DAG with shortcuts:** some transitions skip one or more levels.
→ Solution: introduce **dummy nodes** at intermediate levels to recover a standard lattice structure, then apply the standard [[Bellman Recurrence Relations]].

---

## Examples in this course

| Context | DAG structure |
|---|---|
| [[Dynamic Programming]] | Lattice with N levels |
| [[Edit Distance (Levenshtein)]] | 2D table where $(i+j) = \text{const}$ defines levels |
| [[Dynamic Time Warping]] | 2D alignment grid with monotonicity constraints |
| [[HMM - Decoding (Viterbi Algorithm)]] | Time × states trellis |

---

*Part of → [[MOC - Modelling Sequential Data]]*
