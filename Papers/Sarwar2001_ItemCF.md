---
title: "Item-Based Collaborative Filtering Recommendation Algorithms"
authors: ["Sarwar, Badrul", "Karypis, George", "Konstan, Joseph", "Riedl, John"]
year: 2001
journal: "WWW 2001"
citekey: Sarwar2001
tags: [literature, collaborative-filtering, item-based, recommender-systems, adjusted-cosine]
status: read
---

# Item-Based Collaborative Filtering Recommendation Algorithms

> [!info] Metadata
> **Authors:** Sarwar, Karypis, Konstan, Riedl
> **Year:** 2001 | **Venue:** WWW, pp. 285–295

---

## Abstract

Proposes item-based CF as a scalable alternative to user-based CF. Key insight: item–item similarities are more stable than user–user similarities over time, enabling pre-computation. Introduces **adjusted cosine similarity** (centred on user means) for item–item comparison.

---

## Key Contributions

- **Adjusted cosine** for item similarity: subtracts the user's mean (not the item mean), removing per-user rating-scale bias:

$$\text{sim}(i,j) = \frac{\sum_{u} (r_{ui} - \mu_u)(r_{uj} - \mu_u)}{\sqrt{\sum_u (r_{ui}-\mu_u)^2}\sqrt{\sum_u (r_{uj}-\mu_u)^2}}$$

- Item similarity matrix can be **pre-computed** offline → sub-second online inference
- Empirically outperforms user-based CF at large scale

---

## Relevance to Project

Foundation for `ItemBased` CF model. The adjusted cosine is approximated in the project via TruncatedSVD(k=100) on the item×user matrix for scalability.

---

## Connections

[[Concepts/Collaborative_Filtering]] | [[Papers/Resnick1994_GroupLens]] | [[Codes/04_Models_Collaborative]]
