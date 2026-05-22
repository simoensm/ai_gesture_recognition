---
title: "A sum-over-paths extension of edit distances accounting for all sequence alignments"
authors: ["Garcia-Diez, Sylvain", "Fouss, François", "Shimbo, Masashi", "Saerens, Marco"]
year: 2011
citekey: "GarciaDiez2011_SumOverPaths"
doi: "10.1016/j.patcog.2010.11.020"
venue: "Pattern Recognition, 44(6), 1172–1182"
type: article
tags: [literature, edit-distance, sum-over-paths, sequence-alignment, probabilistic, gesture-recognition, pattern-recognition]
status: read
relevance: high
created: 2026-05-22
---

# A sum-over-paths extension of edit distances accounting for all sequence alignments

> [!info] Metadata
> **Authors:** Sylvain Garcia-Diez, François Fouss, Masashi Shimbo, Marco Saerens
> **Year:** 2011
> **Venue:** Pattern Recognition, 44(6), 1172–1182
> **DOI:** [10.1016/j.patcog.2010.11.020](https://doi.org/10.1016/j.patcog.2010.11.020)

---

## Abstract

Standard edit distance finds the single optimal alignment between two sequences. This paper proposes a **sum-over-paths** generalisation that aggregates over *all* possible alignments, weighted by their cost via a Boltzmann distribution. The resulting dissimilarity measure is smoother, less sensitive to noise, and has a natural probabilistic interpretation.

---

## Key Contributions

### Standard Edit Distance Recap
The classical edit distance selects the minimum-cost alignment path through the DP table:
$$d(s, t) = \min_{\text{alignment}} \sum_{\text{operations}} c(\text{op})$$
Only one path — the optimal — contributes.

### Sum-Over-Paths Generalisation
Instead of taking the minimum, aggregate over all paths $p$ with a Boltzmann weight:
$$d_\beta(s, t) = -\frac{1}{\beta}\ln \sum_{p \in \mathcal{P}(s,t)} e^{-\beta \cdot \text{cost}(p)}$$

where $\beta > 0$ is an inverse temperature parameter:
- $\beta \to \infty$: recovers the standard (minimum) edit distance
- $\beta \to 0$: all paths equally weighted (smoothest, most noise-robust)
- Intermediate $\beta$: balances optimality and robustness

### Probabilistic Interpretation
The weight assigned to path $p$ is:
$$P(p) = \frac{e^{-\beta \cdot \text{cost}(p)}}{\sum_{p'} e^{-\beta \cdot \text{cost}(p')}}$$
This defines a proper probability distribution over alignments. The dissimilarity becomes the **negative log-partition function** — equivalent to computing a free energy in statistical physics.

### Efficient Computation
Computed using a **forward recursion** analogous to the standard DP, but replacing $\min$ with a log-sum-exp operation:
$$Z_\beta(i,j) = \sum_{p \in \mathcal{P}(s_{1:i}, t_{1:j})} e^{-\beta \cdot \text{cost}(p)}$$
Same $\mathcal{O}(nm)$ complexity as standard edit distance.

---

## Relevance to the Gesture Recognition Project

This is a direct extension of the edit distance baseline used in the course project:
- The edit distance approach (Nyirarugira 2014) relies on a **single optimal alignment** — sensitive to quantisation noise in the codebook
- The sum-over-paths measure is **more robust**: a noisy gesture that has a slightly sub-optimal alignment still gets credit from nearby alignments
- $\beta$ can be tuned on validation data — adds one extra hyperparameter but may improve accuracy

**Practical consideration:** The log-sum-exp recursion can have numerical instability; use the standard log-sum-exp trick:
$$\ln\sum_i e^{x_i} = x^* + \ln\sum_i e^{x_i - x^*}, \quad x^* = \max_i x_i$$

---

## Connections

- Extension of → [[Papers/Levenshtein1966|Levenshtein 1966]] and [[Papers/Wagner1974|Wagner & Fischer 1974]]
- Applied to gesture recognition → [[Papers/Nyirarugira2014|Nyirarugira 2014]] (uses standard edit distance)
- Codebook for alphabet → [[Concepts/Vector_Quantization|Vector Quantization]]
- Edit distance concepts → [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance method note]]
- Probabilistic interpretation → [[Foundations/Probability_Theory|Probability Theory]] (Boltzmann distribution)
- DP algorithm → [[Concepts/Dynamic_Programming|Dynamic Programming]]
