---
title: "The Tag Genome: Encoding Community Knowledge to Support Novel Interaction"
authors: ["Vig, Jesse", "Sen, Shilad", "Riedl, John"]
year: 2012
journal: "ACM Transactions on Interactive Intelligent Systems"
citekey: Vig2012
tags: [literature, movielens, tag-genome, content-based, recommender-systems, feature-engineering]
status: read
---

# The Tag Genome: Encoding Community Knowledge to Support Novel Interaction

> [!info] Metadata
> **Authors:** Vig, Sen, Riedl | **Year:** 2012
> **Venue:** ACM TIIS, Vol. 2(3), pp. 1–44

---

## Abstract

Presents the **Tag Genome** — a dataset of continuous relevance scores for 1,128 tags across all MovieLens movies, computed from community-generated tags, ratings, and textual reviews. Enables richer content representations than genre one-hot encoding.

---

## What the Tag Genome Is

For each (movie, tag) pair, the genome provides a **relevance score** ∈ [0, 1]:
- 1.0 = the tag perfectly describes the movie
- 0.0 = completely irrelevant

Examples: ("The Dark Knight", "dark") = 0.98; ("Toy Story", "dark") = 0.12

**Coverage:** 1,128 tags × all MovieLens movies → each movie is a 1,128-dimensional dense vector.

The tags are diverse: mood ("atmospheric", "dark"), theme ("based on a true story"), style ("twist ending"), genre ("sci-fi"), era, etc.

---

## Why It Dominates in the Project

| Feature set | CV RMSE |
|---|---|
| Genre one-hot | 0.8955 |
| Genome SVD (100 dims) | 0.7438 |
| **Genome full (1,128 dims)** | **0.7353** |

The genome encodes richer, more nuanced item descriptions than genre labels. The continuous scores (vs. binary genre flags) allow Ridge regression to learn fine-grained user taste weights.

**Key limitation:** genome coverage is imperfect. Movies with few community tags have sparser, less reliable genome vectors. This is a known weakness for long-tail items.

---

## Connections

[[Concepts/Content_Based_Filtering]] | [[Concepts/TF-IDF]] | [[Papers/Harper2016_MovieLens]] | [[Workflow/04_Feature_Engineering]] | [[Codes/06_Models_Content_Based]]
