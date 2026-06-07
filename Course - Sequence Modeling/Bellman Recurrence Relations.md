---
tags:
  - dynamic-programming
  - bellman
  - recurrence
  - sequence-modeling
---

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

## Proof of the Backward Recurrence

**Goal:** show that $D^*(s_k) = \min_{s_{k+1}}\{d(s_{k+1}\mid s_k) + D^*(s_{k+1})\}$.

**Step 1 — Definition.** By definition, $D^*(s_k)$ is the minimum total cost of the path from $s_k$ to the end:

$$D^*(s_k) = \min_{(s_{k+1},\dots,s_N)} \sum_{i=k+1}^{N} d(s_i \mid s_{i-1})$$

**Step 2 — Peel off the first transition.** The first term of the sum is $d(s_{k+1}\mid s_k)$, which depends only on $s_{k+1}$, so factor it out:

$$D^*(s_k) = \min_{(s_{k+1},\dots,s_N)} \left\{ d(s_{k+1} \mid s_k) + \sum_{i=k+2}^{N} d(s_i \mid s_{i-1}) \right\}$$

**Step 3 — Split the minimisation.** Because $d(s_{k+1}\mid s_k)$ does not depend on $(s_{k+2},\dots,s_N)$, we can split the joint minimisation into two nested ones:

$$D^*(s_k) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + \min_{(s_{k+2},\dots,s_N)} \sum_{i=k+2}^{N} d(s_i \mid s_{i-1}) \right\}$$

**Step 4 — Recognise the sub-problem.** The inner minimisation is exactly the definition of $D^*(s_{k+1})$:

$$\min_{(s_{k+2},\dots,s_N)} \sum_{i=k+2}^{N} d(s_i \mid s_{i-1}) = D^*(s_{k+1})$$

**Conclusion.** Substituting gives the Bellman recurrence:

$$\boxed{D^*(s_k) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + D^*(s_{k+1}) \right\}}$$

This is valid for all $k = N-1, N-2, \dots, 0$, with boundary condition $D^*(s_N) = 0$ (no cost once the end is reached). The key principle is the **principle of optimality**: the tail of an optimal path is itself optimal, which is what allows the split in Step 3.

---

*Part of → [[Dynamic Programming]] | [[MOC - Modelling Sequential Data]]*
