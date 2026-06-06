---
title: "GroupLens: An Open Architecture for Collaborative Filtering of Netnews"
authors: ["Resnick, Paul", "Iacovou, Neophytos", "Suchak, Mitesh", "Bergstrom, Peter", "Riedl, John"]
year: 1994
journal: "CSCW 1994"
citekey: Resnick1994
tags: [literature, collaborative-filtering, recommender-systems, history, Usenet]
status: read
---

# GroupLens: An Open Architecture for Collaborative Filtering of Netnews

> [!info] Metadata
> **Authors:** Resnick et al. | **Year:** 1994 | **Venue:** CSCW, pp. 175–186

---

## Abstract

Introduces the term **collaborative filtering** and presents GroupLens — the first deployed CF system, built for filtering Usenet newsgroup articles. Users rate articles; the system finds users with similar ratings and recommends articles liked by similar users.

---

## Key Contributions

- Coins the term "collaborative filtering"
- First working deployment of a CF recommender
- Demonstrates that aggregated user opinions can filter information overload
- Introduces the basic user-based CF pipeline: collect ratings → find neighbours → predict → recommend

---

## Historical Significance

GroupLens (1994) → MovieLens (1997, same research group at Minnesota) → Harper & Konstan (2016) [[Papers/Harper2016_MovieLens]]. This paper is the direct ancestor of the dataset used in the MLSMM2156 project.

---

## Connections

[[Concepts/Collaborative_Filtering]] | [[Papers/Sarwar2001_ItemCF]] | [[Papers/Harper2016_MovieLens]]
