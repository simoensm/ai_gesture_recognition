---
tags: [NLP, text-mining, information-retrieval, feature-extraction, recommender-systems]
---
# TF-IDF (Term Frequency – Inverse Document Frequency)

**TF-IDF** is a numerical statistic that reflects how important a word is to a document in a corpus. It is the standard vectorisation method for converting free text into feature vectors for machine learning.

**See also:** [[Concepts/Content_Based_Filtering]] | [[Papers/Salton1988_TFIDF]]

---

## Formula

$$\text{TF-IDF}(w, d, D) = \underbrace{\frac{f_{w,d}}{\sum_{w'} f_{w',d}}}_{\text{TF}} \times \underbrace{\log\frac{|D|}{|\{d' \in D : w \in d'\}|}}_{\text{IDF}}$$

**TF (Term Frequency):** how often term $w$ appears in document $d$, normalised by document length. Rewards terms frequent _in this document_.

**IDF (Inverse Document Frequency):** penalises terms that appear in many documents (uninformative). A term in every document gets IDF ≈ 0; a rare term gets high IDF.

**Result:** high TF-IDF = term appears often in this document but rarely across the corpus → highly discriminative.

---

## Text Mining Pipeline

```
Raw Text
   ↓  Tokenisation
["High-tech", "multiverse", "where", "humans", …]
   ↓  Stop-word Removal
["High-tech", "multiverse", "humans", "greatest", …]
   ↓  Stemming / Lemmatisation
["hightech", "multiverse", "human", "great", "innov", …]
   ↓  Frequency Filtering  (remove very rare / very common tokens)
["hightech", "human", "great", "dark"]
   ↓  TF-IDF Vectorisation
{"hightech": 0.88, "human": 0.62, "great": 0.20, "dark": 0.90}
```

---

## Alternative: LDA (Latent Dirichlet Allocation)

TF-IDF produces **sparse high-dimensional vectors** (one dimension per word). LDA reduces text to a small set of latent topics (dense, low-dimensional), enabling explainability ("Because you like Sci-Fi") and handling synonymy.

Each document = mixture of topics; each topic = distribution over words.

Python: `from gensim.models import LdaModel`

---

## In Recommender Systems

In content-based filtering, TF-IDF converts movie descriptions/tags into item vectors $\mathbf{x}_i$. The user profile $\mathbf{w}_u$ learns which TF-IDF dimensions matter to the user. Cosine similarity between profile and item vectors ranks recommendations.

For MovieLens, the **genome tag scores** (continuous relevance per tag per movie) are conceptually similar to TF-IDF but computed by crowd-sourcing rather than term frequency. See [[Papers/Vig2012_TagGenome]].

---

## Properties

- **Sparse:** most terms get weight 0 for most documents
- **Unsupervised:** no labels needed
- **Bag-of-words:** word order ignored
- **Scale-free:** normalised by document length

**Limitations:** ignores word order (syntax), polysemy (same word, different meanings), synonymy (different words, same meaning). LDA partially addresses the latter two.
