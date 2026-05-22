
Tags: #dynamic-programming #bellman #recurrence

---

## Context

These recurrences are the core computation of [[Dynamic Programming]]. They allow computing the optimal cost efficiently by reusing already-computed subproblem solutions.

---

## Backward Recurrence (standard)

Start from the **last level** and work backwards:

$$D^*(s_N) = 0$$

$$D^*(s_k = i) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k = i) + D^*(s_{k+1}) \right\}$$

$$D^* = \min_{s_0} \{ D^*(s_0) \}$$

**Interpretation:** To find the optimal move from state $i$ at level $k$, consider each possible transition to level $k+1$, add its immediate cost to the already-known optimal future cost $D^*(s_{k+1})$, then take the minimum.

---

## Forward Recurrence (symmetric)

Start from the **first level** and work forward:

$$D^*(s_0) = 0$$

$$D^*(s_k = i) = \min_{s_{k-1}} \left\{ d(s_k = i \mid s_{k-1}) + D^*(s_{k-1}) \right\}$$

$$D^* = \min_{s_N} \{ D^*(s_N) \}$$

---

## General Form (with shortcut links / DAG)

When jumps can skip levels (see [[Directed Acyclic Graph (DAG)]]):

$$D^*(s_k = i) = \min_{\{s\} \mid s_k = i \to s} \left\{ d(s \mid s_k = i) + D^*(s) \right\}$$

where $\{s\} \mid s_k = i \to s$ is the **set of successor states** of $i$.

---

## Key Insight (Proof Sketch)

The recurrence works because the minimum can be factored:

$$D^*(s_k) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + \underbrace{\min_{s_{k+2},\dots,s_N} \sum_{i=k+2}^{N} d(s_i \mid s_{i-1})}_{D^*(s_{k+1})} \right\}$$

---

*Part of → [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
