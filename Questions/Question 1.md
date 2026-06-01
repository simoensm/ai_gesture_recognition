---
tags:
  - dynamic-programming
  - recurrence
---
>Explain and derive in detail (mathematically) the dynamic programming recurrence formula. In addition, what are the different applications of dynamic programming that were introduced in the course ?

---

## Answer

### What is Dynamic Programming?

Dynamic programming (DP) is an optimisation paradigm for problems with **optimal substructure**: the optimal solution can be built from optimal solutions to sub-problems. It avoids recomputation by solving each sub-problem once.

The problem must be representable as a **Directed Acyclic Graph (DAG)** with **levels** $k = 0,\dots,N$ and **states** $s_k$. Moving from $s_{k-1}$ to $s_k$ incurs cost $d(s_k \mid s_{k-1})$.

---

### Setup

The goal is to find the path minimising total cost:

$$D^* = \min_{(s_1,\dots,s_N)} \sum_{i=1}^{N} d(s_i \mid s_{i-1})$$

Define the **optimal cost-to-go** from state $s_k$:

$$D^*(s_k) = \min_{(s_{k+1},\dots,s_N)} \sum_{i=k+1}^{N} d(s_i \mid s_{i-1})$$

with boundary condition $D^*(s_N) = 0$.

---

### Derivation of the Backward Bellman Recurrence

**Step 1 — Definition:**

$$D^*(s_k) = \min_{(s_{k+1},\dots,s_N)} \sum_{i=k+1}^{N} d(s_i \mid s_{i-1})$$

**Step 2 — Peel off the first transition:**

$$= \min_{(s_{k+1},\dots,s_N)} \left\{ d(s_{k+1} \mid s_k) + \sum_{i=k+2}^{N} d(s_i \mid s_{i-1}) \right\}$$

**Step 3 — Split the minimisation** ($d(s_{k+1}\mid s_k)$ does not depend on $s_{k+2},\dots,s_N$):

$$= \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + \min_{(s_{k+2},\dots,s_N)} \sum_{i=k+2}^{N} d(s_i \mid s_{i-1}) \right\}$$

**Step 4 — Recognise the sub-problem:**

$$= \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + D^*(s_{k+1}) \right\}$$

**Conclusion:**

$$\boxed{D^*(s_k) = \min_{s_{k+1}} \left\{ d(s_{k+1} \mid s_k) + D^*(s_{k+1}) \right\}}$$

This is the **principle of optimality**: the tail of an optimal path is itself optimal, which justifies the split in Step 3. The recurrence is computed for $k = N-1, N-2, \dots, 0$, and $D^* = \min_{s_0} D^*(s_0)$.

---

### Complexity

- Naïve search: $\mathcal{O}(n^N)$ — exponential
- Dynamic programming: $\mathcal{O}(n^2 \cdot N)$ — polynomial (each of the $n \times N$ states computed once, with $n$ successors each)

---

### Applications Introduced in the Course

**1. Edit Distance (Levenshtein distance)**
Compare two strings $\mathbf{x}, \mathbf{y}$ via insertion, deletion, substitution. State = $(i,j)$, level = $i+j$:

$$D^*(i,j) = \min \begin{cases} D^*(i,j-1)+1 \\ D^*(i-1,j)+1 \\ D^*(i-1,j-1)+\delta_{ij} \end{cases}$$

**2. Longest Common Subsequence (LCS)**
Related to edit distance: $\text{lcs}(\mathbf{x},\mathbf{y}) = \frac{1}{2}(|\mathbf{x}|+|\mathbf{y}|-\text{dist}(\mathbf{x},\mathbf{y}))$ when substitution penalty = 2.

**3. Dynamic Time Warping (DTW)**
Optimal non-linear alignment of two speech signals. State = $(i,j)$, cost = frame distance $d(r_i, o_j)$:

$$g(i,j) = \min \begin{cases} g(i-1,j)+d(r_i,o_j) \\ g(i-1,j-1)+2\,d(r_i,o_j) \\ g(i,j-1)+d(r_i,o_j) \end{cases}$$

**4. Expected number of visits before absorption (Fundamental Matrix)**
$\mathbf{N} = (\mathbf{I}-\mathbf{Q})^{-1} = \sum_{t=0}^{\infty}\mathbf{Q}^t$ counts expected visits to transient states.

**5. Expected cost before absorption**
$V(i) = \sum_{k \in \text{Succ}(i)} p_{ik}(c_{ik} + V(k))$ — direct Bellman recurrence on the absorbing chain.

**6. HMM Forward Algorithm**
$\alpha_j(t+1) = \left(\sum_i \alpha_i(t)p_{ij}\right) b_j(x_{t+1})$ — DP on the HMM trellis to compute $P(\mathbf{x}\mid\theta)$.

**7. Viterbi Algorithm**
Most likely state sequence in an HMM:
$\delta_j(t+1) = \max_i\{\delta_i(t)\cdot p_{ij}\cdot b_j(x_{t+1})\}$ — DP with max instead of sum.
