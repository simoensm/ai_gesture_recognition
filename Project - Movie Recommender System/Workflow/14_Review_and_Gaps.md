---
tags: [recommender-systems, project-review, gaps, analysis]
---
# Project Review — Gaps and Improvement Points

Critical review of the project's strengths, gaps, and open questions.

**Related:** [[Workflow/08_Results_and_Decision_Making]] | [[Workflow/09_Beyond_Accuracy]] | [[Workflow/07_Statistical_Assessment]]

---

## 1. Rich Data Not Fully Used

The ablation CSVs contain `ild`, `coverage`, `novelty`, `fit_time`, `pred_time` for every configuration. Key findings from the actual numbers:

| Family | RMSE | ILD | Coverage | Novelty |
|---|---|---|---|---|
| ContentBased (best) | 0.734 | **0.662** | 5.6% | 5.4 |
| SVD (best) | 0.791 | 0.734 | 6.0% | 4.0 |
| NMF f=10 | 0.823 | **0.751** | 2.2% | 9.5 |
| UserBased Spearman | 0.834 | 0.733 | 3.7% | 9.5 |

**Coverage is catastrophically low (2–8%) across all models** — the system recommends the same tiny slice of the catalogue. NMF, despite being weakest on RMSE, produces the most diverse recommendations. This **accuracy–diversity trade-off** is the core analytical story.

---

## 2. Models Designed but Never Evaluated

- **ItemBased CF** — present in `configs.py` section 1c, zero results in any CSV
- **SVDpp** — commented out, never run
- **HybridSVD** — class built and described in theory, but never in the ablation experiments

---

## 3. No Statistical Significance Testing (full scope)

RMSE differences between top models are in the 0.001–0.01 range. A paired Wilcoxon test across 5 folds (as done in gesture recognition) would confirm whether ContentBased is _significantly_ better than SVD, or whether the difference is fold-sampling noise.

---

## 4. Inference Time Never Discussed

From the CSVs:
- ContentBased fits in **8–25 s per fold** (depending on regressor)
- SVD fits in **1.8 s**
- KNNWithMeans fits in **0.2–0.5 s**

A 20-second wait to fit a Ridge regressor with 1,128 features per request is a real production concern, never mentioned despite the webapp implying fast recommendations.

---

## 5. Analytics Notebook Invisible in Report

`01_analytics.ipynb` characterises the rating distribution, genre distribution, long-tail, sparsity, and user activity. None of those findings appear in the data section. The 97% sparsity with a heavily long-tailed item distribution is precisely _why_ content-based with genome tags dominates — popular items have rich genome vectors while CF fails on tail items due to sparse co-ratings.

---

## 6. No Cold-Start Quantification

Cold start is mentioned as a limitation but never measured. A simple experiment — subsample each user's ratings to 1, 3, 5, 10, 20 ratings and plot RMSE vs. number of ratings — would directly quantify the cold-start boundary and justify the "≥3 ratings" claim in the webapp.

---

## 7. No Error Analysis

Which users have the worst predictions? Power users or sparse users? Which movies are systematically under/over-predicted? Are the errors concentrated on specific genres?

---

## 8. Apples-and-Oranges RMSE Comparisons

Some RMSE values in `configs.py` are annotated as single 75/25 split, others as 5-fold CV. A split RMSE is not comparable to a CV RMSE and cannot appear in the same summary table.

---

## 9. Hackathon — Missing Actual Leaderboard Scores

The leakage problem is well-explained but the actual submitted scores are never stated. What was the leaderboard RMSE before and after fixing the leakage?

---

## 10. References Thin for Master Level

Nine references, three of which are tool citations. Missing: Koren 2008 for SVDpp, Ricci et al. 2011 RecSys survey, standard bias-variance references.

---

## Random Seed — Three Places, Three Reasons

**1. `KFold(random_state=RANDOM_SEED)`** — Fixes which ratings go into which fold. Without it, models would be evaluated on different splits, making comparisons unfair.

**2. `random.Random(RANDOM_SEED).sample(uids, max_users)`** — Fixes which users are sampled for ILD/Coverage/Novelty. Without it, model A might be evaluated on easy-to-recommend users while model B gets niche users.

**3. `rng = random.Random(RANDOM_SEED)` in `_partial_anti_testset`** — Fixes which unrated items are included in the anti-testset for the diversity pass.

The principle is **reproducibility**: same seed → byte-identical results → fair comparisons → reproducible findings.

---

## Webapp Flow Clarification

1. `MovieContext.jsx` POSTs to `http://localhost:5001/recommend` with the user's ratings, watched history, and onboarding preferences.
2. `server.py` runs the chosen model — `fit()` + `predict()` on the fly for content-based, fold-in via Ridge for SVD — and returns `{recommendations: [...], matchScores: {...}}`.
3. The **only client-side modification** on top of server scores is the mood adjustment (±10 pts max) and the genre boost cap (+15). The underlying ranking is always the model's.
4. The pure client-side genre-boost heuristic is a **fallback** that only activates when `serverOnline === false`.

---

## Open TODOs

- [ ] Hybrid SVD to ablation and statistical test
- [ ] Add bar chart: best RMSE per family
- [ ] Add Ridge α sweep: U-shaped regularization curve
- [ ] Add SVD++ sweep: RMSE vs n_factors line plot
- [ ] Add ILD vs RMSE scatter: diversity-accuracy trade-off plot
- [ ] Genome > TMDB but what happens for movies with **no genome coverage**?
- [ ] Leakage fix for item stats scaled
- [ ] Update README
- [ ] ITR and AMI = stats?
- [ ] Random vs temporal splitting?
