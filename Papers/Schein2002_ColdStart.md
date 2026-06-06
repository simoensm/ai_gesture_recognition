---
title: "Methods and Metrics for Cold-Start Recommendations"
authors: ["Schein, Andrew I.", "Popescul, Alexandrin", "Ungar, Lyle H.", "Pennock, David M."]
year: 2002
journal: "SIGIR 2002"
citekey: Schein2002
tags: [literature, cold-start, recommender-systems, evaluation, hybrid]
status: read
doi: "10.1145/564376.564421"
---

# Methods and Metrics for Cold-Start Recommendations

> [!info] Metadata
> **Authors:** Schein et al. | **Year:** 2002 | **Venue:** SIGIR, pp. 253–260

---

## Abstract

Proposes a PLSI-based approach to cold-start recommendations (new users or new items with no ratings). Introduces metrics for evaluating cold-start performance and shows that content information dramatically improves recommendations in the cold-start regime.

---

## Cold-Start Problem

**New user cold start:** a new user has no rating history → CF similarity is undefined. The system must rely on:
- Onboarding preferences (stated preferences)
- Demographic data
- Content-based fallback (no ratings needed)

**New item cold start:** a new item has no ratings → CF cannot include it in recommendations. Content-based approaches handle this naturally since only item attributes are needed.

---

## Quantifying Cold Start

A cold-start experiment: subsample user ratings to 1, 3, 5, 10, 20 ratings and plot RMSE vs. number of ratings. The "knee" in the curve defines the cold-start boundary.

---

## Relevance to Project

Justification for the webapp's Phase 0/1 cascade: content-based cold-start before transitioning to the collaborative model at ≥20 ratings. The paper supports the claim that content information is critical for cold-start users.

The 3-rating minimum for Phase 1 is somewhat arbitrary — this paper suggests a principled experiment would quantify the actual breakeven point. (Listed as an open TODO in [[Workflow/14_Review_and_Gaps]].)

---

## Connections

[[Concepts/Content_Based_Filtering]] | [[Papers/Burke2002_HybridRS]] | [[Webapp/03_Cascade_Architecture]]
