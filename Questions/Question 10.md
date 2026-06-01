---
tags:
  - markov
  - markov-chain
  - markov-models
  - absorbing
---
>Explain the general principles and the structure of a hidden Markov model (HMM) for speech recognition. Moreover, derive in detail (mathematically) how we can solve the first important problem in HMMs, namely computing the likelihood of the observations (the probability of generating a sequence of observations). Finally, how can we recognize an uttered word among a finite dictionary of words.

---

## Answer

### General Principles and Structure of an HMM

A **Hidden Markov Model (HMM)** extends a Markov chain by distinguishing between **hidden states** (not directly observable) and **observations** (emitted by the states). It is well-suited for speech recognition because:
- The acoustic signal is observable but the underlying phonemes/words are not
- The same word can be pronounced at different speeds — the hidden states absorb this variability

**Model parameters** $\theta = \{\mathbf{\Pi}, \mathbf{P}, \mathbf{B}\}$:

| Parameter | Symbol | Definition |
|---|---|---|
| Initial state distribution | $\mathbf{\Pi}$ | $\pi_i = P(s_1=i)$ |
| Transition matrix | $\mathbf{P}$ | $p_{ij} = P(s_{t+1}=j\mid s_t=i)$ |
| Emission probabilities | $\mathbf{B}$ | $b_i(o_k) = P(x_t=o_k\mid s_t=i)$ |

**Assumptions:**
1. **Markov property:** $P(s_{t+1}\mid s_1,\dots,s_t) = P(s_{t+1}\mid s_t)$
2. **Output independence:** $P(x_t\mid s_1,\dots,s_T,x_1,\dots,x_T) = P(x_t\mid s_t)$ — the observation at time $t$ depends only on the current hidden state

**In speech recognition:**
- One HMM per reference word in the dictionary
- Each state corresponds to a phonetic unit or sub-word unit
- Observations = spectrogram frames (feature vectors extracted from audio)

---

### Problem 1: Computing the Likelihood $P(\mathbf{x}\mid\theta)$

Given observation sequence $\mathbf{x} = [x_1,\dots,x_T]$ and model $\theta$, compute $P(\mathbf{x}\mid\theta)$.

#### Decomposition

**Step 1 — Observation probability given states** (independence assumption):

$$P(\mathbf{x}\mid\mathbf{s},\theta) = \prod_{t=1}^{T} b_{s_t}(x_t)$$

**Step 2 — State sequence probability** (Markov property):

$$P(\mathbf{s}\mid\theta) = \pi_{s_1}\prod_{t=1}^{T-1} p_{s_t s_{t+1}}$$

**Step 3 — Joint probability:**

$$P(\mathbf{x},\mathbf{s}\mid\theta) = P(\mathbf{x}\mid\mathbf{s},\theta)\,P(\mathbf{s}\mid\theta)$$

**Step 4 — Marginal likelihood** (sum over all state sequences):

$$P(\mathbf{x}\mid\theta) = \sum_{\mathbf{s}} P(\mathbf{x}\mid\mathbf{s},\theta)\,P(\mathbf{s}\mid\theta)$$

#### Naïve Approach (intractable)

This sum runs over all combinations $s_1\in\{1,\dots,n\},\dots,s_T\in\{1,\dots,n\}$ — giving $n^T$ state sequences. Complexity $\mathcal{O}(n^T\cdot T)$ — completely intractable for realistic values.

#### Efficient Solution: The Forward Algorithm

Exploit the DAG (trellis) structure via dynamic programming.

**Define the forward variable:**

$$\alpha_i(t) = P(x_1\dots x_t,\ s_t=i\mid\theta)$$

= joint probability of observing $x_1,\dots,x_t$ **and** being in state $i$ at time $t$.

**Initialisation** ($t=1$):

$$\alpha_i(1) = \pi_i\,b_i(x_1)$$

**Induction** ($t=1,\dots,T-1$):

$$\alpha_j(t+1) = \left(\sum_{i=1}^{n}\alpha_i(t)\,p_{ij}\right)b_j(x_{t+1})$$

Each $\alpha_j(t+1)$ accumulates all paths arriving at state $j$ at time $t+1$, weighted by the emission probability.

**Likelihood:**

$$\boxed{P(\mathbf{x}\mid\theta) = \sum_{j=1}^{n}\alpha_j(T)}$$

**Complexity:** $\mathcal{O}(n^2 T)$ — polynomial. For $n=5$, $T=100$: $2500$ operations vs $5^{100}\approx 10^{69}$ for the naïve approach.

---

### The Backward Algorithm (complementary)

**Define the backward variable:**

$$\beta_i(t) = P(x_{t+1}\dots x_T\mid s_t=i,\theta)$$

= probability of future observations given current state $i$.

**Initialisation:** $\beta_i(T) = 1\ \forall i$

**Induction** ($t=T-1,\dots,1$):

$$\beta_i(t) = \sum_{j=1}^{n}p_{ij}\,b_j(x_{t+1})\,\beta_j(t+1)$$

**Alternative likelihood:**

$$P(\mathbf{x}\mid\theta) = \sum_{i=1}^{n}\pi_i\,\beta_i(1) = \sum_{i=1}^{n}\alpha_i(t)\,\beta_i(t)\quad\forall t$$

The last identity uses the key factorisation:

$$P(\mathbf{x},s_t=i\mid\theta) = \alpha_i(t)\,\beta_i(t)$$

which holds by the Markov property: given $s_t=i$, past and future observations are independent.

---

### Word Recognition

Given a finite dictionary of $W$ reference words, each with its own HMM $\theta^{(w)}$, and an observed utterance $\mathbf{x}$:

1. For each word $w$, compute $P(\mathbf{x}\mid\theta^{(w)})$ using the Forward Algorithm.
2. Select the word with **maximum likelihood**:

$$\hat{w} = \arg\max_{w} P(\mathbf{x}\mid\theta^{(w)})$$

By Bayes' rule, if all words are equally likely a priori, this is equivalent to maximum a posteriori (MAP) decoding.

This is analogous to the DTW nearest-neighbour approach but replaces the deterministic distance with a probabilistic likelihood — handling variability in pronunciation more flexibly.
