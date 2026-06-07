---
tags:
  - dynamic-programming
  - optimisation
  - sequence-modeling
---

Tags: #dynamic-programming #optimization #shortest-path

---

## Definition

Dynamic programming (DP) solves optimization problems on a **lattice** (a graph organized in levels) by breaking them into smaller overlapping subproblems.

The canonical problem: find the **minimum-cost path** from level 0 to level N.
→ This is a **shortest-path problem**.

---

## Key Concepts


- **State** $s_k$: the variable holding the current node at level $k$
- **Local cost** $d(s_k = j \mid s_{k-1} = i)$: cost of transitioning from state $i$ at level $k-1$ to state $j$ at level $k$
- **Total cost** of a path $(s_0, s_1, \dots, s_N)$:

$$D(s_0, s_1, \dots, s_N) = \sum_{i=1}^{N} d(s_i \mid s_{i-1})$$

- **Optimal cost** from initial state $s_0$:

$$D^*(s_0) = \min_{(s_1, \dots, s_N)} \left\{ \sum_{i=1}^{N} d(s_i \mid s_{i-1}) \right\}$$

---

## Principle of Optimality (Bellman)

The optimal cost from any intermediate state $s_k$ only depends on that state — not on how we arrived at it.

See → [[Bellman Recurrence Relations]]

---

## Finding the Optimal Path

To retrieve the **optimal path**, not just the cost:
1. **Store a pointer** at each node pointing to the optimal predecessor
2. **Backtrack** from the final node to the initial node

---

## Applicable Structures

DP can be applied to any **[[Directed Acyclic Graph (DAG)]]**.

For graphs with **shortcut links** (skipping levels), introduce **dummy nodes** to recover a standard level structure.

---

## Applications in this course

| Application                            | Notes                            |
| -------------------------------------- | -------------------------------- |
| [[Edit Distance (Levenshtein)]]        | DP on a 2D character table       |
| [[Dynamic Time Warping]]               | DP on a signal alignment lattice |
| [[HMM - Decoding (Viterbi Algorithm)]] | DP on the HMM trellis            |
| [[HMM - Forward Procedure]]            | DP forward recursion             |
| [[HMM - Backward Procedure]]           | DP backward recursion            |

---

*Part of → [[MOC - Modelling Sequential Data]]*
