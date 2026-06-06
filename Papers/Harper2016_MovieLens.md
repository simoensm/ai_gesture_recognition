---
title: "The MovieLens Datasets: History and Context"
authors: ["Harper, F. Maxwell", "Konstan, Joseph A."]
year: 2016
journal: "ACM Transactions on Interactive Intelligent Systems"
citekey: Harper2016
tags: [literature, movielens, dataset, recommender-systems, benchmark]
status: read
---

# The MovieLens Datasets: History and Context

> [!info] Metadata
> **Authors:** Harper, Konstan | **Year:** 2016
> **Venue:** ACM TIIS, Vol. 5(4), pp. 19:1–19:19

---

## Abstract

Describes the history, construction, and context of the MovieLens datasets — the dominant benchmark for recommender systems research. Covers the MovieLens website, data collection methodology, dataset variants (100K, 1M, 10M, 20M, 25M), and guidance on appropriate use.

---

## Key Facts — MovieLens 1M (used in project)

| Property | Value |
|---|---|
| Users | ~6,040 |
| Movies | ~3,952 |
| Ratings | 1,000,209 |
| Rating scale | 1–5 (integer) |
| Density | ~4.2% |
| Mean rating | ~3.58 |

---

## Dataset Splits and Usage Notes

- Each user rated at least 20 movies
- Ratings include timestamp → enables temporal splitting
- No explicit demographic data beyond age/gender/occupation (anonymised)

**Important:** the 1M dataset is sparse enough that most CF methods struggle. This is precisely why content-based with genome features dominates in the MLSMM2156 project.

---

## Connections

[[Workflow/02_Data]] | [[Papers/Vig2012_TagGenome]] | [[Concepts/Recommender_Systems]]
