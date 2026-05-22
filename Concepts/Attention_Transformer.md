---
title: "Attention Mechanisms & Transformers"
tags: [concept, attention, transformer, self-attention, bert, gpt, nlp, deep-learning]
created: 2026-05-22
---

# Attention Mechanisms & Transformers

> [!abstract] The paradigm shift
> "Attention is All You Need" (Vaswani et al., 2017) replaced recurrent processing with parallelisable self-attention. Transformers now dominate NLP, vision, and increasingly time-series and gesture recognition.

**Key papers:** [[Papers/Vaswani2017_Transformer|Vaswani 2017]], [[Papers/Devlin2018_BERT|Devlin 2018 (BERT)]]

---

## 1. Scaled Dot-Product Attention

Given queries $Q$, keys $K$, values $V$ (all $\in \mathbb{R}^{n \times d}$):

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$$

- $QK^\top$: similarity scores between all query-key pairs — $\mathcal{O}(n^2 d)$
- $\sqrt{d_k}$: scaling to prevent softmax saturation in high dimensions
- softmax: convert scores to weights (sum to 1)
- Weighted sum of values: the output is a mixture of values, weighted by query-key similarity

**Interpretation:** Each query "attends to" all keys, extracts a weighted average of the corresponding values. Allows any position to directly attend to any other — no recurrence needed.

---

## 2. Multi-Head Attention

Run $h$ attention heads in parallel with different projections:

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$
$$\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$$

Each head learns to attend to different aspects: syntax, semantics, coreference, etc.

---

## 3. Transformer Architecture

```
Input Embeddings + Positional Encoding
        │
        ▼
[Encoder Block] × N:
    Multi-Head Self-Attention
    Add & Norm (residual)
    Feed-Forward Network (FFN)
    Add & Norm

        │ (for seq2seq; encoder-only for BERT, decoder-only for GPT)
        ▼

[Decoder Block] × N:
    Masked Multi-Head Self-Attention
    Cross-Attention (attends to encoder)
    FFN
    Add & Norm
        │
        ▼
    Output probabilities
```

**Key components:**
- **Positional encoding:** $PE_{(pos, 2i)} = \sin(pos/10000^{2i/d})$ — injects position information (attention is permutation-invariant)
- **FFN:** Two linear layers with GELU/ReLU between: $\text{FFN}(x) = W_2 \max(0, W_1 x + b_1) + b_2$
- **Residual connections + LayerNorm** in every block (stabilises training)

---

## 4. Self-Attention vs. Recurrence

| Property | RNN/LSTM | Transformer |
|---|---|---|
| **Parallelism** | Sequential (step by step) | Fully parallel |
| **Long-range deps** | Difficult (vanishing gradient) | Direct (any distance, $\mathcal{O}(1)$ path) |
| **Memory** | Fixed hidden state | Grows with sequence length |
| **Complexity** | $\mathcal{O}(nd^2)$ | $\mathcal{O}(n^2 d)$ per layer |
| **Best for** | Short sequences, online | Long sequences, offline |

For gesture sequences of length 64: LSTM and Transformer are comparable; Transformer has advantage on longer sequences.

---

## 5. BERT vs. GPT

| | BERT (Devlin et al., 2018) | GPT (Radford et al., 2018) |
|---|---|---|
| **Architecture** | Encoder only | Decoder only |
| **Attention** | Bidirectional (full sequence) | Causal (left-to-right only) |
| **Pre-training** | Masked LM + Next Sentence Prediction | Language modelling |
| **Best for** | Understanding, classification | Generation, completion |
| **Fine-tuning** | Add task head, fine-tune all | Prompt / in-context learning |

---

## 6. Attention for Time Series & Gestures

Attention can be applied to gesture sequences:
- Replace BiLSTM with Transformer encoder
- Self-attention captures non-local temporal dependencies
- Attention weights become interpretable: which frames were most discriminative?

Our BiLSTM already uses **attention pooling** as an optional mode: `pooling="attention"` — see [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]].

---

## Connections

- Replaces → [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]] for long sequences
- Position encoding is analogous to temporal indexing in gestures
- Key paper: [[Papers/Vaswani2017_Transformer|Attention is All You Need]]
- BERT pre-training: [[Papers/Devlin2018_BERT|BERT]]
- Mathematical foundation: [[Foundations/Linear_Algebra_for_ML|Linear Algebra]] (QKV projections)
