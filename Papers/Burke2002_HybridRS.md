---
title: "Hybrid Recommender Systems: Survey and Experiments"
authors: ["Burke, Robin"]
year: 2002
journal: "User Modeling and User-Adapted Interaction"
citekey: Burke2002
tags: [literature, hybrid, recommender-systems, cold-start, collaborative-filtering, content-based]
status: read
doi: "10.1023/A:1021240730564"
---

# Hybrid Recommender Systems: Survey and Experiments

> [!info] Metadata
> **Authors:** Robin Burke | **Year:** 2002
> **Venue:** UMUAI, Vol. 12(4), pp. 331–370

---

## Abstract

A comprehensive survey and experimental comparison of hybrid recommender system architectures. Identifies seven hybridisation strategies and evaluates them on the EntreeC restaurant recommendation dataset.

---

## Seven Hybridisation Strategies

| Strategy | Description |
|---|---|
| **Weighted** | Linear combination of scores from multiple recommenders |
| **Switching** | Choose one recommender based on a criterion (e.g., number of ratings) |
| **Mixed** | Present recommendations from multiple sources side by side |
| **Feature combination** | Use output of one system as features for another |
| **Cascade** | One system refines the output of another |
| **Feature augmentation** | One system generates features used as input to another |
| **Meta-level** | One model learned by one is used as input to another |

---

## Relevance to Project

The webapp implements a **cascade hybrid** (Phase 0/1/2) — exactly Burke's cascade strategy, where a content-based cold-start model progressively gives way to the configured warm-start model as the user provides more ratings.

The `HybridSVD` model uses a **weighted** combination: $\hat{r} = \alpha \cdot \hat{r}_\text{content} + (1-\alpha) \cdot \hat{r}_\text{SVD}$.

See [[Webapp/03_Cascade_Architecture]] | [[Codes/07_Models_Hybrid]]

---

## Connections

[[Concepts/Recommender_Systems]] | [[Concepts/Collaborative_Filtering]] | [[Concepts/Content_Based_Filtering]] | [[Papers/Schein2002_ColdStart]]
