---
title: "Evaluating Collaborative Filtering Recommender Systems"
authors: ["Herlocker, Jonathan L.", "Konstan, Joseph A.", "Terveen, Loren G.", "Riedl, John"]
year: 2004
journal: "ACM Transactions on Information Systems"
citekey: Herlocker2004
tags: [literature, evaluation, recommender-systems, RMSE, hit-rate, collaborative-filtering]
status: read
doi: "10.1145/963770.963772"
---

# Evaluating Collaborative Filtering Recommender Systems

> [!info] Metadata
> **Authors:** Herlocker, Konstan, Terveen, Riedl | **Year:** 2004
> **Venue:** ACM TOIS, Vol. 22(1), pp. 5–53

---

## Abstract

A comprehensive survey of evaluation methodologies for collaborative filtering systems. Addresses the choice of metrics (accuracy, coverage, novelty, serendipity), evaluation protocols (offline vs. user studies), and data splitting strategies.

---

## Key Points

**RMSE vs MAE:** The paper argues RMSE is preferred over MAE as it penalises large errors more, which is important in recommendation (a very bad recommendation is worse than a mediocre one).

**Coverage:** Fraction of user-item pairs for which the system can make a prediction. Low coverage = the system refuses to recommend on many queries.

**Novelty:** Recommending items the user hasn't seen before (distinct from "non-obvious"). 

**Serendipity:** Surprising yet relevant recommendations — the hardest metric to capture offline.

**Offline vs. user study:** Offline evaluation (RMSE, hit rate) correlates imperfectly with user satisfaction. LOO hit rate is a proxy for ranking quality but cannot measure serendipity.

---

## Relevance to Project

The standard reference for the evaluation choices made in MLSMM2156 (RMSE as primary metric, LOO hit rate for ranking, ILD/Coverage/Novelty for beyond-accuracy). The paper's note that coverage/novelty "cannot be obtained offline" is partially addressed by our anti-testset diversity metrics.

---

## Connections

[[Workflow/03_Split_and_Evaluation]] | [[Workflow/09_Beyond_Accuracy]] | [[Concepts/Recommender_Systems]] | [[Papers/Cremonesi2010_TopN]]
