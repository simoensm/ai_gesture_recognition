### 1. You have rich data you are not using

Your ablation CSVs contain `ild`, `coverage`, `novelty`, `fit_time`, `pred_time` for every configuration. The report barely touches these. Looking at the actual numbers:

|Family|RMSE|ILD|Coverage|Novelty|
|---|---|---|---|---|
|ContentBased (best)|0.734|**0.662**|5.6%|5.4|
|SVD (best)|0.791|0.734|6.0%|4.0|
|NMF f=10|0.823|**0.751**|2.2%|9.5|
|UserBased Spearman|0.834|0.733|3.7%|9.5|

**Coverage is catastrophically low across all models (2–8%).** This means your system is essentially recommending the same tiny slice of the catalogue to everyone. This is the single most important finding the report does not discuss. NMF, despite being the weakest on RMSE, produces the most diverse recommendations by ILD. This accuracy–diversity trade-off is the core analytical story missing from the results section.

### 2. Models that were designed but never evaluated

- **ItemBased CF** — in `configs.py` section 1c, zero results in any CSV
- **SVDpp** — commented out, never run
- **HybridSVD** — four commented lines in configs.py, zero results. You built the class, you described it in the theory section, but there is no experiment

For a Master-level project these gaps matter. Either run them and report, or explicitly justify why they were excluded.

### 3. No statistical significance testing

The RMSE differences between your top models are in the 0.001–0.01 range. You do not know whether these are statistically meaningful or just fold-sampling noise. A paired Wilcoxon signed-rank test across the 5 folds (exactly as your gesture recognition report did) would tell you whether ContentBased is _significantly_ better than SVD, or whether the difference could be random. Without this, the ranking in Table 4 of the report is an observation, not a finding.

### 4. Inference time is never discussed — and it matters

From the CSVs: ContentBased fits in **8–25 seconds per fold** (depending on regressor). SVD fits in **1.8 seconds**. KNNWithMeans fits in **0.2–0.5 seconds**. In a webapp serving real users, a 20-second wait to fit a Ridge regressor with 1,128 features per request is a real problem. This is never mentioned in the report, and the webapp section implies recommendations are fast.

### 5. Analytics notebook is invisible in the report

`01_analytics.ipynb` presumably characterises the rating distribution, genre distribution, long-tail, sparsity, and user activity. None of those findings appear in the data description section. The data section currently states numbers without showing what they imply (e.g., 97% sparsity with a heavily long-tailed item distribution is exactly _why_ content-based with genome tags dominates — popular items have rich genome vectors while CF fails on tail items due to sparse co-ratings).

### 6. No learning curve / cold-start quantification

You mention cold start as a limitation but never measure it. A simple experiment — subsample each user's ratings to 1, 3, 5, 10, 20 ratings and plot RMSE vs. number of ratings — would directly quantify the cold-start boundary and justify the "≥3 ratings" claim in the webapp section.

### 7. No error analysis

Which users have the worst predictions? Power users (many ratings) or sparse users? Which movies are systematically under/over-predicted? Are the errors concentrated on specific genres? This is what separates a measurement project from an analysis project.

### 8. The report compares apples and oranges

Some RMSE values in `configs.py` are annotated as single 75/25 split, others as 5-fold CV. For example `genre_genome_weighted` has `RMSE=0.7571` (split) while the best model has `CV RMSE=0.7344`. A split RMSE is not comparable to a CV RMSE and cannot appear in the same summary table.

### 9. Hackathon section lacks actual leaderboard scores

The section explains the leakage problem well but never states what your submitted scores were. "Results were slower" is vague — what was the actual leaderboard RMSE before and after fixing the leakage?

### 10. References are thin for a Master level

Nine references, three of which are tool citations (Surprise, scikit-learn, GitHub). No reference for Ridge regression (Hoerl & Kennard 1970 is already in your `.bib`), no reference for the bias-variance decomposition, no RecSys survey (Ricci et al. 2011 is the standard). Missing Koren 2008 for SVDpp since you mention it.



Random seed : Three places, three different reasons:

**1. `KFold(random_state=RANDOM_SEED)`** — the most important one. Fixes which ratings go into which fold. Without it, every run would produce different train/test splits, making it impossible to compare models — model A might have gotten "easier" users in its test set than model B purely by chance.

**2. `random.Random(RANDOM_SEED).sample(uids, max_users)`** (diversity users) Fixes _which_ users are sampled for ILD/Coverage/Novelty. Without it, model A might be evaluated on 300 easy-to-recommend users while model B gets 300 niche users — a fake difference in diversity scores.

**3. `rng = random.Random(RANDOM_SEED)` in `_partial_anti_testset`** Same idea — fixes which unrated items are included in the anti-testset for the diversity pass.

---

The principle is **reproducibility**: given the same seed, two runs produce byte-identical results. This means:

- You can re-run a single model config weeks later and get the same RMSE
- Comparisons between models are fair (same splits, same sampled users)
- Your paper/report results are reproducible by someone else
- 42 is just a convention — any fixed integer works equally well