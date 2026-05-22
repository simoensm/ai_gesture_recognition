# HMM - Applications

Tags: #HMM #applications #NLP #speech #bioinformatics

---

## 1. Speech Recognition

### Isolated Word Recognition
- Each reference word in a dictionary = one [[Hidden Markov Model (HMM)]]
- States = phonemes (sounds like /p/, /a/, /r/…)
- Observations = acoustic feature vectors (from spectrograms)
- Approach: compute $P(\mathbf{x} \mid \theta^{(w)})$ for each word $w$, select highest → same idea as [[Dynamic Time Warping]] but probabilistic

### Connected Speech
- Stochastic grammars are added on top of HMMs to handle word sequences

---

## 2. Part-of-Speech (POS) Tagging

Label each word in a sentence with its grammatical role.

**Example:**
> "The representative put chairs on the table"
> → AT NN VBD NNS IN AT NN

| Tag | Meaning |
|---|---|
| AT | Article |
| NN | Singular noun |
| VBD | Verb, past tense |
| NNS | Plural noun |
| IN | Preposition |
| JJ | Adjective |

**HMM setup:**
- Hidden states = POS tags
- Observations = words
- Decoding via [[HMM - Decoding (Viterbi Algorithm)]]

---

## 3. Bioinformatics

Reference: *Durbin et al., Biological Sequence Analysis*, Cambridge University Press.

**Application: Protein secondary structure prediction**
- Observations (primary structure): amino acid sequence (e.g. ATCPLELLLD)
- Hidden states (secondary structure): structural labels (e.g. HHHBBBBBC)
  - H = alpha-helix
  - B = beta-sheet
  - C = coil

---

*Part of → [[Hidden Markov Model (HMM)]] | [[MOC - Modelling Sequential Data]]*
