---
title: "Fundamentals of Speech Recognition"
authors: ["Rabiner, Lawrence R.", "Juang, Biing-Hwang"]
year: 1993
citekey: "Rabiner1993_SpeechRecognition"
doi: ""
venue: "Prentice-Hall"
type: book
tags: [literature, speech-recognition, hmm, dtw, dynamic-programming, feature-extraction, sequence-modelling, pattern-recognition]
status: read
relevance: high
created: 2026-05-22
---

# Fundamentals of Speech Recognition

> [!info] Metadata
> **Authors:** Lawrence R. Rabiner, Biing-Hwang Juang (AT&T Bell Labs)
> **Year:** 1993
> **Publisher:** Prentice-Hall
> **ISBN:** 9780130151575

---

## Abstract

The canonical reference for speech recognition, covering the full pipeline from acoustic features to language models. Particularly influential for its rigorous treatment of Dynamic Time Warping and Hidden Markov Models as the two dominant paradigms for sequence alignment and classification. The HMM chapter is the standard reference for Baum-Welch and Viterbi derivations.

---

## Key Contributions

### Chapter 4 — Dynamic Time Warping

- Formalises DTW for speech: $D(i,j) = d(x_i, y_j) + \min(D(i-1,j), D(i,j-1), D(i-1,j-1))$
- Introduces **slope constraints** (Sakoe-Chiba band, Itakura parallelogram) to prevent degenerate alignments
- Endpoint handling: global vs. open-ended (subsequence) DTW
- Weighted symmetric and asymmetric path step patterns

### Chapter 6 — Hidden Markov Models

The most cited chapter — derives the three HMM algorithms from scratch:

**Forward algorithm:** $\alpha_t(j) = b_j(o_t)\sum_i \alpha_{t-1}(i)a_{ij}$

**Backward algorithm:** $\beta_t(i) = \sum_j a_{ij}b_j(o_{t+1})\beta_{t+1}(j)$

**Baum-Welch (EM):** Re-estimation formulas for $\hat{A}$, $\hat{B}$, $\hat{\pi}$ using $\gamma$ and $\xi$ quantities.

**Viterbi:** Max-product algorithm — dynamic programming through the trellis.

### Feature Extraction (Chapter 3)

Covers acoustic features — MFCC (Mel-Frequency Cepstral Coefficients) for speech. The principles transfer directly to gesture recognition:
- Windowing + feature extraction → replaces raw audio with velocity/acceleration features
- Delta and delta-delta features → capture temporal dynamics (analogous to gesture kinematics)

---

## DTW vs. HMM for Sequence Recognition

| Aspect | DTW | HMM |
|---|---|---|
| **Model** | Non-parametric (uses stored templates) | Parametric (learned transition & emission) |
| **Training** | No training needed | Requires labelled sequences + EM |
| **Variability** | Limited (one template per class) | Handles intra-class variation |
| **Decision** | Distance to nearest template | Likelihood $P(O\mid\lambda)$ |
| **Reference** | [[Papers/Sakoe1978\|Sakoe & Chiba 1978]] | [[Papers/Fink2014_MarkovModels\|Fink 2014]] |

The HMM approach is more powerful but requires enough training data for Baum-Welch convergence. With 10 training sequences per class (Huang dataset), DTW is competitive.

---

## Relevance to Project

- The slope constraints described in Chapter 4 are directly applicable to the Sakoe-Chiba band in our DTW baseline
- Chapter 6 provides the derivations referenced when justifying why BiLSTM may be preferred over HMM for small datasets (BiLSTM does not require explicit state topology)
- Feature extraction insights inform preprocessing choices for the gesture sequences

---

## Connections

- DTW algorithm → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW method note]] → [[Papers/Sakoe1978|Sakoe & Chiba 1978]]
- HMM treatment → [[Papers/Fink2014_MarkovModels|Fink 2014]] (more modern, gesture-focused)
- DP foundations → [[Concepts/Dynamic_Programming|Dynamic Programming]]
- EM algorithm → [[Concepts/Clustering_EM|Clustering & EM Algorithm]]
- Sequence modelling context → [[Papers/Hochreiter1997|LSTM]] as successor
