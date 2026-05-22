# Hidden Markov Model (HMM)

Tags: #HMM #generative-model #sequential-data #probabilistic

---

## Definition

A **Hidden Markov Model (HMM)** is a **graphical generative model** for sequential data where:
- The underlying state sequence is **hidden** (unobserved)
- Only a sequence of **observations** is visible
- The hidden states follow the [[Markov Property]]

---

## Graphical Structure

```
s₁ → s₂ → s₃ → ... → sT
↓    ↓    ↓           ↓
x₁   x₂   x₃  ...    xT
```

- **Green nodes** = hidden states $s_t \in \{1, \dots, n\}$
- **Purple nodes** = observations $x_t \in \{o_1, \dots, o_p\}$
- Arrows = probabilistic dependencies

---

## Parameters → see [[HMM Formalism]]

| Symbol | Name | Definition |
|---|---|---|
| $\mathbf{\Pi} = \{\pi_i\}$ | Initial state probs | $P(s_1 = i)$ |
| $\mathbf{P} = \{p_{ij}\}$ | Transition probs | $P(s_{t+1}=j \mid s_t=i)$ |
| $\mathbf{B} = \{b_i(o_k)\}$ | Emission probs | $P(x_t=o_k \mid s_t=i)$ |

---

## The Three Core Problems

| Problem | Algorithm | Goal |
|---|---|---|
| 1. **Likelihood** | [[HMM - Forward Procedure]] | $P(\mathbf{x} \mid \theta)$ |
| 2. **Decoding** | [[HMM - Decoding (Viterbi Algorithm)]] | $\arg\max_\mathbf{s} P(\mathbf{s} \mid \mathbf{x}, \theta)$ |
| 3. **Learning** | [[HMM - Parameter Estimation (Baum-Welch)]] | $\arg\max_\theta P(\mathbf{x} \mid \theta)$ |

---

## Example: Word as HMM

Each word is modelled as a sequence of **phonemes** (sounds):
$$P \to A \to R \to I \to S$$

Each phoneme = a hidden state, emitting acoustic feature vectors.

---

## Applications → [[HMM - Applications]]

- Speech recognition
- Part-of-speech tagging
- Bioinformatics sequence modelling

---

*Part of → [[MOC - Modelling Sequential Data]]*
