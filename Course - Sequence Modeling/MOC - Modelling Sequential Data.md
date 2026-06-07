---
tags:
  - MOC
  - sequence-modeling
  - index
---
# 🗺️ MOC — Modelling Sequential Data
**Course:** Machine Learning — Marco Saerens (UCLouvain)

This is the entry point of the vault. Every concept links to a dedicated note.

---

## 📦 Main Topics

### 1. Dynamic Programming & Edit Distance
- [[Dynamic Programming]] — Shortest-path on a lattice, Bellman recurrences
- [[Bellman Recurrence Relations]] — Backward & forward recurrences
- [[Directed Acyclic Graph (DAG)]] — Structure required for DP
- [[Edit Distance (Levenshtein)]] — String comparison via DP
- [[Editing Operations]] — Insertion, Deletion, Substitution
- [[Longest Common Subsequence]] — Derived from edit distance
- [[Dynamic Time Warping]] — Signal alignment via DP

### 2. Markov Chains
- [[Markov Chain]] — Core definition, states, transitions
- [[Transition Probability Matrix]] — Stochastic matrix P
- [[Markov Property]] — Memoryless assumption
- [[Stationary Distribution]] — Long-run behaviour π
- [[Absorbing Markov Chain]] — Absorbing & transient states
- [[Fundamental Matrix]] — N = (I−Q)⁻¹, expected visits
- [[Absorption Probabilities]] — B = NR matrix
- [[Expected Cost Until Absorption]] — V(i) computation
- [[Irreducible and Regular Markov Chains]] — Convergence conditions

### 3. Applications of Markov Chains
- [[Application - Customer Lifetime Value]] — Marketing application
- [[Application - ItemRank (Collaborative Filtering)]] — Random walk with restart

### 4. Hidden Markov Models (HMM)
- [[Hidden Markov Model (HMM)]] — Generative model definition
- [[HMM Formalism]] — Parameters Π, P, B
- [[HMM - Likelihood Computation]] — Forward procedure
- [[HMM - Forward Procedure]] — α variables
- [[HMM - Backward Procedure]] — β variables
- [[HMM - Decoding (Viterbi Algorithm)]] — Most likely state sequence
- [[HMM - Parameter Estimation (Baum-Welch)]] — EM algorithm
- [[HMM - Applications]] — Speech, NLP, bioinformatics

---

## 🔗 Key Algorithmic Connections

| Algorithm | Technique | Problem |
|---|---|---|
| Edit Distance | DP on 2D table | String similarity |
| Dynamic Time Warping | DP on lattice | Signal alignment |
| Viterbi | DP on DAG | HMM decoding |
| Forward/Backward | DP recursion | HMM likelihood |
| Baum-Welch | EM algorithm | HMM learning |

---

*Tags: #sequential-data #markov #HMM #dynamic-programming*
