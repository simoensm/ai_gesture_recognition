---
title: "Markov Models for Pattern Recognition: From Theory to Applications"
authors: ["Fink, Gernot A."]
year: 2014
citekey: "Fink2014_MarkovModels"
doi: "10.1007/978-1-4471-6308-4"
venue: "Springer (2nd edition)"
type: book
tags: [literature, hmm, markov-models, pattern-recognition, sequence-modelling, speech-recognition, gesture-recognition, viterbi, baum-welch]
status: read
relevance: high
created: 2026-05-22
---

# Markov Models for Pattern Recognition: From Theory to Applications

> [!info] Metadata
> **Authors:** Gernot A. Fink (TU Dortmund)
> **Year:** 2014 (2nd edition)
> **Publisher:** Springer
> **DOI:** [10.1007/978-1-4471-6308-4](https://doi.org/10.1007/978-1-4471-6308-4)

---

## Abstract

A comprehensive treatment of Hidden Markov Models (HMMs) and their applications to pattern recognition, with a particular focus on sequence data (speech, gestures, handwriting). Covers discrete and continuous HMMs, model topology choices, the three fundamental HMM algorithms, and practical issues in training and decoding.

---

## Key Concepts

### Hidden Markov Model Definition

An HMM $\lambda = (A, B, \pi)$ consists of:
- **$A$** — Transition matrix: $a_{ij} = P(q_{t+1} = j \mid q_t = i)$
- **$B$** — Emission probabilities: $b_j(o_t) = P(o_t \mid q_t = j)$
- **$\pi$** — Initial state distribution: $\pi_i = P(q_1 = i)$

The hidden state sequence $q_1, q_2, \ldots, q_T$ is unobservable; only the observation sequence $O = o_1, o_2, \ldots, o_T$ is seen.

### The Three Fundamental Problems

| Problem | Question | Algorithm |
|---|---|---|
| **Evaluation** | $P(O \mid \lambda)$ — how likely is this sequence? | Forward algorithm |
| **Decoding** | $\arg\max_Q P(Q \mid O, \lambda)$ — what state sequence? | Viterbi algorithm |
| **Learning** | $\arg\max_\lambda P(O \mid \lambda)$ — fit the model to data | Baum-Welch (EM) |

### Forward Algorithm
$$\alpha_t(j) = P(o_1,\ldots,o_t, q_t=j \mid \lambda) = b_j(o_t)\sum_i \alpha_{t-1}(i) a_{ij}$$
Final probability: $P(O\mid\lambda) = \sum_i \alpha_T(i)$. Complexity: $\mathcal{O}(N^2 T)$.

### Viterbi Algorithm
$$\delta_t(j) = \max_{q_1,\ldots,q_{t-1}} P(q_1,\ldots,q_{t-1},q_t=j,o_1,\ldots,o_t \mid \lambda)$$
Backtrack through argmax pointers to recover the most likely path. Also $\mathcal{O}(N^2 T)$.

### Baum-Welch (EM for HMMs)

E-step: compute $\gamma_t(i)$ (prob. of being in state $i$ at time $t$) and $\xi_t(i,j)$ (prob. of transitioning $i \to j$ at time $t$) using forward-backward algorithm.

M-step: re-estimate:
$$\hat{a}_{ij} = \frac{\sum_t \xi_t(i,j)}{\sum_t \gamma_t(i)}, \quad \hat{\pi}_i = \gamma_1(i)$$

### Continuous Observation HMMs
Replace discrete emission $b_j(o)$ with a Gaussian (or Gaussian Mixture Model):
$$b_j(o) = \sum_m c_{jm} \mathcal{N}(o; \mu_{jm}, \Sigma_{jm})$$
GMM emissions are the standard in speech and gesture recognition.

---

## HMM Topologies for Gestures

- **Left-to-right (Bakis):** States advance left to right only ($a_{ij} = 0$ for $j < i$) — models temporal progression of a gesture
- **Ergodic:** Any state can follow any other — for general pattern recognition
- **Left-to-right with skips:** Allows skipping states — accommodates variable-speed execution

---

## Relationship to Other Methods

| Method | HMM connection |
|---|---|
| DTW | HMM decoding without learned emissions — both align query to template |
| Viterbi | Dynamic programming — same DP structure as DTW |
| Baum-Welch | EM algorithm applied to latent state sequences |
| BiLSTM | Learns implicit state representations rather than explicit Markov structure |

---

## Connections

- EM algorithm → [[Concepts/Clustering_EM|Clustering & EM Algorithm]]
- Compared to DTW → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW]]
- Speech application → [[Papers/Rabiner1993_SpeechRecognition|Rabiner & Juang 1993]]
- Sequence modelling → [[Papers/Hochreiter1997|LSTM]] vs. HMM approaches
- Dynamic programming (Viterbi) → [[Concepts/Dynamic_Programming|Dynamic Programming]]
- Probability foundations → [[Foundations/Probability_Theory|Probability Theory]]
