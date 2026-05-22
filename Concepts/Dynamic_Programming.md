---
title: "Dynamic Programming"
tags:
  - concept
  - dynamic-programming
  - algorithms
  - optimization
created: 2026-05-20
---

# Dynamic Programming

> [!abstract] One-line definition
> A method for solving complex optimization problems by breaking them into overlapping subproblems and storing the results of subproblems to avoid redundant computation.

---

## Core Idea

Dynamic programming (DP) exploits **optimal substructure** and **overlapping subproblems**:

1. **Optimal substructure:** The optimal solution to the problem can be built from optimal solutions to its subproblems
2. **Overlapping subproblems:** The same subproblems recur many times → store results in a table (memoization or bottom-up tabulation)

**General recurrence pattern:**
$$\text{DP}(i) = f\left(\text{DP}(i-1), \text{DP}(i-2), \ldots, x_i\right)$$

---

## Instances in This Project

### DTW (Dynamic Time Warping)
$$D(i, j) = C(i, j) + \min\{D(i-1, j),\ D(i, j-1),\ D(i-1, j-1)\}$$
- Table: $n \times m$ cost matrix
- Goal: minimum-cost warping path from $(1,1)$ to $(n,m)$
- See: [[Papers/Sakoe1978]], [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|01_DTW_Dynamic_Time_Warping]]

### Edit Distance (Levenshtein)
$$\text{lev}(i,j) = \min\{\text{lev}(i-1,j)+1,\ \text{lev}(i,j-1)+1,\ \text{lev}(i-1,j-1)+[s_i \neq t_j]\}$$
- Same $n \times m$ table structure as DTW
- See: [[Papers/Levenshtein1966]], [[Papers/Wagner1974]], [[Project - Gesture Recognition/02_Edit_Distance|02_Edit_Distance]]

### [[Concepts/Longest_Common_Subsequence|Longest Common Subsequence (LCS)]]
$$\text{LCS}(i,j) = \begin{cases} \text{LCS}(i-1,j-1)+1 & \text{if } s_i = t_j \\ \max(\text{LCS}(i-1,j),\ \text{LCS}(i,j-1)) & \text{otherwise} \end{cases}$$
- Third baseline option (alongside DTW and edit distance)
- See: [[Papers/Theodoridis2009]]

### Bellman Recurrence (Markov Chains)
Used in absorbing Markov chain analysis in the [[Course - Sequence Modeling/MOC - Modelling Sequential Data|Sequence Modeling course]].

---

## Complexity

All three sequence-comparison DP algorithms share:
- **Time:** $\mathcal{O}(nm)$ where $n, m$ are sequence lengths
- **Space:** $\mathcal{O}(nm)$ for full table; $\mathcal{O}(\min(n,m))$ with rolling array trick
- **With Sakoe-Chiba band:** $\mathcal{O}(rn)$ time for DTW

---

## Key Insight for This Project

DTW, edit distance, and LCS are all **instances of the same DP template** applied to different cost functions. Understanding one makes the others immediate. The table size and step pattern differ, but the memoization structure is identical.
