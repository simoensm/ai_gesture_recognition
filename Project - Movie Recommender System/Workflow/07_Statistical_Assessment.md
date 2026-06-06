# Statistical Assessment

Can we trust that RMSE differences between models reflect genuine quality, not fold-sampling luck?

**File:** `recsys/statistical_assessment.py` | Code details: [[Codes/08_Ablation_and_Assessment_Code]]

---

## The Problem

We have fold-level RMSE values for each model. A difference of 0.005 in mean RMSE might be real, or it might come from a lucky/unlucky fold split. Statistical tests tell us whether observed differences are **statistically significant**.

---

## Why Non-Parametric Tests?

We **cannot use a simple t-test** because:
1. RMSE values are bounded (can't be negative) → not normally distributed
2. With only 3–5 folds, we can't verify normality
3. We have **paired observations**: all models are evaluated on the *same* folds

The Friedman test is the non-parametric equivalent of a repeated-measures ANOVA, exploiting the paired structure without assuming normality.

---

## Step 1: Friedman Test (Omnibus)

**Question:** Are all models equivalent, or does at least one model differ?

**How it works:**
1. For each fold, **rank** all models from best (rank 1) to worst (rank K), based on RMSE (lower RMSE = rank 1)
2. Average ranks across folds → mean rank per model
3. Compute the Friedman statistic:

$$\chi^2_F = \frac{12N}{k(k+1)} \sum_{j=1}^{k} \left(\bar{r}_j - \frac{k+1}{2}\right)^2$$

where N = number of folds, k = number of models, r̄ⱼ = mean rank of model j.

4. Compare to chi-squared distribution with k−1 degrees of freedom

**H₀:** All models are equivalent (their fold-level RMSE distributions come from the same population).  
If p < 0.05, we reject H₀ and conclude **at least one model is different**.

---

## Step 2: Post-Hoc Tests

The Friedman test only says "something is different." To find *which pairs* differ, we run post-hoc tests.

### Nemenyi Test (Demšar 2006)
Uses the **Critical Difference (CD)**:

$$\text{CD} = q_\alpha \sqrt{\frac{k(k+1)}{6N}}$$

where qα is a lookup value based on number of models k (e.g. q₀.₀₅ = 2.343 for k=3).

Two models are **significantly different** if |mean_rank_A − mean_rank_B| ≥ CD.

Visualised as a **Critical Difference Diagram** (Demšar 2006): models connected by a bar are *not* significantly different.

### Wilcoxon Signed-Rank + Holm-Bonferroni
Pairwise Wilcoxon tests on raw per-fold RMSE differences (uses the magnitude, not just ranks → more powerful than Nemenyi when folds are adequate).

With multiple comparisons, p-values are corrected using **Holm-Bonferroni**: sort p-values ascending; the i-th is tested at α/(k−i).

---

## Results

### Cross-Family Comparison (best model per family)

| Model | Avg Rank | Mean RMSE |
|---|---|---|
| **Hybrid (α=0.2, genome full + SVD)** | **1.0** | **0.7308** |
| SVD++ f=150, λ=0.05, η=0.01 | 2.0 | 0.7890 |
| ItemBased MSD k=40 | 3.0 | 0.8254 |

**Friedman:** χ² = 6.00, p = 0.050 (borderline significant — limited power from 3-fold constraint)  
**Nemenyi (k=3, N=3, CD=1.91):**
- Hybrid vs. ItemBased CF: |rank diff| = 2.0 ≥ CD → **significant** (p=0.038)
- Hybrid vs. SVD++: |rank diff| = 1.0 < CD → not significant (p=0.44)

**Conclusion:** The hybrid model is statistically significantly better than item-based CF. The gap with SVD++ is consistent but not statistically provable with 3 folds.

---

### Within Content-Based Family

| Model | Avg Rank | Mean RMSE |
|---|---|---|
| Genre one-hot | 5.0 | 0.8955 |
| Genome SVD (n=100) | 4.0 | 0.7438 |
| Genome full, RidgeCV | 2.8 | 0.7360 |
| Genome full, Ridge α=20 | 1.6 | 0.7353 |
| **Genome full + stats, Ridge α=20** | **1.6** | **0.7346** |

**Friedman:** χ² = 17.92, p = 0.00128 → **highly significant**  
**Nemenyi (k=5, N=5, CD=2.73):** Genre one-hot vs. genome full: |4.0−1.6| = 3.4 ≥ 2.73 → **significant**  
→ The genome jump is statistically confirmed.

---

### Within Latent Factor Family

| Model | Avg Rank | Mean RMSE |
|---|---|---|
| NMF f=10 | 5.0 | 0.8229 |
| SVD default (λ=0.02) | 4.0 | 0.8148 |
| SVD λ=0.05, η=0.005 | 3.0 | 0.7976 |
| SVD++ f=100, λ=0.05, η=0.01 | 2.0 | 0.7904 |
| **SVD f=100, λ=0.05, η=0.01** | **1.0** | **0.7913** |

**Friedman:** χ² = 20.00, p = 0.00050 → **highly significant**  
**Nemenyi:**
- Tuned SVD vs. NMF: significant (p=0.0006)
- Tuned SVD vs. default SVD: significant (p=0.023)
- Tuned SVD vs. SVD++: **not significant** (p=0.856)

> The tuned SVD ranks first despite being slightly worse in mean RMSE than SVD++ (0.7913 vs 0.7890). This is because SVD++ has higher fold-to-fold variance — SVD ranks better in enough individual folds to win the rank average.

---

### Within Hybrid Family

| Model | Avg Rank | Mean RMSE |
|---|---|---|
| Hybrid α=0.2 | 1.0 | 0.7308 |
| Hybrid α=0.1 | 2.2 | 0.7316 |
| … | … | … |
| Hybrid α=0.9 | 9.0 | 0.7776 |

**Friedman:** χ² = 39.79, p = 0.000004 → **highly significant**  
**Nemenyi (CD=5.37):** α ≤ 0.2 are significantly better than α ≥ 0.7

The SVD weight α has a **monotonic, statistically significant** effect on RMSE.

---

## Five Key Statistical Conclusions

1. **The hybrid model is the overall best** — statistically confirmed vs. item-based CF.
2. **SVD++ matches tuned SVD statistically** — but both are significantly better than defaults.
3. **Genome features drive content performance** — the genome vs. genre gap is highly significant.
4. **Hyperparameter tuning is a first-order effect** — λ sweep saves more than model-family switching.
5. **Hybrid blending confirms orthogonal signals** — α drives a significant monotonic effect.

---

> [!WARNING]+ ❓ Assessor Question — Why use the Friedman test instead of multiple t-tests?
> Running k(k-1)/2 pairwise t-tests inflates the Type-I error rate (false positive rate). If we run 10 tests at α=0.05, we expect ~0.5 false positives by chance. The Friedman test is an **omnibus** test: it tests all models simultaneously, controlling the overall error rate at 0.05. Only after the Friedman test is significant do we run pairwise post-hoc tests (with correction for multiple comparisons via Holm-Bonferroni or Nemenyi).

> [!WARNING]+ ❓ Assessor Question — What does the Critical Difference (CD) represent?
> CD is the minimum difference in mean rank needed for two models to be considered significantly different. If model A has mean rank 1.0 and model B has mean rank 3.1, and CD=2.73, then |3.1 − 1.0| = 2.1 < 2.73 → **not significant**. CD depends on k (more models → higher CD, harder to detect differences), N (more folds → smaller CD, easier to detect), and α (significance level). In other words, more models = harder to declare any pair as different; more folds = easier.

> [!WARNING]+ ❓ Assessor Question — Why is the cross-family test borderline significant (p=0.050)?
> Item-based CF can only run 3-fold CV (not 5-fold) due to its O(|I|²) similarity matrix computation time. This forces the cross-family Friedman test to use only 3 folds for all models (the intersection). With N=3 folds and k=3 models, the Friedman test has very low statistical power — it can barely detect even large differences. The p=0.050 result is a direct consequence of this power limitation, not evidence that the hybrid is only marginally better.

---

**See also:** [[Workflow/06_Ablation_Study]] | [[Codes/08_Ablation_and_Assessment_Code]] | [[Workflow/08_Results_and_Decision_Making]]
