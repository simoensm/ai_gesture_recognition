---
title: "Attention Is All You Need"
authors: ["Vaswani, Ashish", "Shazeer, Noam", "Parmar, Niki", "Uszkoreit, Jakob", "Jones, Llion", "Gomez, Aidan N.", "Kaiser, Łukasz", "Polosukhin, Illia"]
year: 2017
citekey: "Vaswani2017_Transformer"
doi: "10.48550/arXiv.1706.03762"
venue: "NeurIPS 2017"
type: conference
tags: [literature, transformer, attention, self-attention, nlp, deep-learning, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# Attention Is All You Need

> [!info] Metadata
> **Authors:** Ashish Vaswani et al. (Google Brain / Google Research)
> **Year:** 2017
> **Venue:** NeurIPS 2017
> **arXiv:** [1706.03762](https://arxiv.org/abs/1706.03762)

---

## Abstract

Introduces the **Transformer architecture**, dispensing with recurrence and convolutions entirely in favour of self-attention. Achieves state-of-the-art translation quality while being more parallelisable and faster to train. This paper is arguably the most influential in modern AI — BERT, GPT, and all large language models are built on this architecture.

---

## Key Contributions

- **Scaled dot-product attention:** $\text{Attention}(Q,K,V) = \text{softmax}(QK^\top/\sqrt{d_k})V$
- **Multi-head attention:** $h$ parallel attention heads, each with its own projections
- **Positional encoding:** sinusoidal or learned position vectors
- **Encoder-Decoder architecture:** encoder (bidirectional) + decoder (causal)
- Fully **parallelisable** — unlike RNNs, no sequential computation
- $\mathcal{O}(n^2 d)$ complexity per layer but excellent GPU utilisation

## Results

- 28.4 BLEU on WMT 2014 English-to-German (new SOTA)
- 41.0 BLEU on English-to-French
- Trained in 12 hours on 8 GPUs (vs. weeks for previous RNN models)

## Critical Analysis

**Strengths:**
- Parallelism enables training on massive datasets
- Quadratic attention is optimal for capturing all pairwise dependencies
- Spawned BERT, GPT, T5, GPT-3, GPT-4, LLaMA, and the entire LLM era

**Limitations:**
- $\mathcal{O}(n^2)$ memory for long sequences
- Position encoding doesn't fully capture ordering (unlike RNNs)
- Requires large datasets and compute to shine

## Connections

- Architecture explained in → [[Concepts/Attention_Transformer|Attention & Transformers]]
- Compared to → [[Papers/Hochreiter1997|LSTM / BiLSTM]]
- Pre-training extensions → [[Papers/Devlin2018_BERT|BERT]], GPT
- Applied to vision → ViT (Dosovitskiy et al., 2020)
- Could replace BiLSTM in gesture classification for longer sequences
