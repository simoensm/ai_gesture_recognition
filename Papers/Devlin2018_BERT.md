---
title: "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding"
authors: ["Devlin, Jacob", "Chang, Ming-Wei", "Lee, Kenton", "Toutanova, Kristina"]
year: 2018
citekey: "Devlin2018_BERT"
doi: "10.48550/arXiv.1810.04805"
venue: "NAACL 2019"
type: conference
tags: [literature, bert, transformer, pre-training, fine-tuning, nlp, bidirectional, masked-lm, deep-learning]
status: read
relevance: high
created: 2026-05-22
---

# BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding

> [!info] Metadata
> **Authors:** Jacob Devlin, Ming-Wei Chang, Kenton Lee, Kristina Toutanova (Google AI)
> **Year:** 2018
> **Venue:** NAACL 2019
> **arXiv:** [1810.04805](https://arxiv.org/abs/1810.04805)

---

## Abstract

Introduces **BERT** (Bidirectional Encoder Representations from Transformers), a language model pre-trained on massive unlabelled text using two objectives: **Masked Language Modelling (MLM)** and **Next Sentence Prediction (NSP)**. BERT's bidirectional context (unlike GPT's left-to-right decoder) achieved new SOTA on 11 NLP benchmarks simultaneously.

---

## Key Contributions

- **Masked Language Model (MLM):** Randomly mask 15% of tokens; predict the masked token using the full left+right context — enables truly bidirectional representations
- **Next Sentence Prediction (NSP):** Binary task: does sentence B follow sentence A? — captures inter-sentence reasoning
- **Pre-train → Fine-tune paradigm:** Pre-train once on a large corpus; fine-tune cheaply on any downstream task with a single additional layer
- **Architecture:** Standard Transformer encoder (12 or 24 layers), $d_{model} = 768$ or 1024, trained on BooksCorpus + Wikipedia

## Two Model Variants

| | BERT-Base | BERT-Large |
|---|---|---|
| **Layers** | 12 | 24 |
| **Hidden size** | 768 | 1024 |
| **Attention heads** | 12 | 16 |
| **Parameters** | 110M | 340M |

## Results

- GLUE benchmark: 80.5% → new record at the time
- SQuAD 1.1 F1: 93.2 (human: 91.2 — BERT surpassed human performance)
- 11 NLP tasks improved simultaneously

## Pre-train vs. Fine-tune vs. Feature Extraction

- **Fine-tuning:** Add a task head on top of `[CLS]` token; update all weights on task-specific labelled data
- **Feature extraction (ELMo-style):** Freeze BERT, use contextual embeddings as features
- Fine-tuning generally outperforms feature extraction

## Connections

- Built on → [[Concepts/Attention_Transformer|Transformer architecture]] (encoder only)
- Compared to → [[Papers/Vaswani2017_Transformer|Vaswani et al. 2017]] (original Transformer)
- GPT uses the decoder (causal), BERT uses the encoder (bidirectional)
- Foundation for RoBERTa, ALBERT, DistilBERT, domain-specific BERTs
- Applied to sequence labelling → could classify gesture segments
