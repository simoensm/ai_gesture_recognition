---
title: "Statistical Comparisons of Classifiers over Multiple Data Sets"
authors: ["Demšar, Janez"]
year: 2006
citekey: "Demsar2006_StatisticalComparisons"
doi: "10.5555/1248547.1248548"
venue: "Journal of Machine Learning Research, 7, 1–30"
type: article
tags: [literature, statistical-tests, hypothesis-testing, classifier-comparison, wilcoxon, friedman, nemenyi, evaluation, foundational]
status: read
relevance: high
created: 2026-05-22
---

# Statistical Comparisons of Classifiers over Multiple Data Sets

> [!info] Metadata
> **Authors:** Janez Demšar (University of Ljubljana)
> **Year:** 2006
> **Venue:** Journal of Machine Learning Research, 7, 1–30
> **Free PDF:** [jmlr.org/papers/v7/demsar06a.html](https://jmlr.org/papers/v7/demsar06a.html)

---

## Abstract

The definitive reference for statistically comparing machine learning classifiers. Argues against the widely-used paired t-test (which violates normality assumptions on accuracy data) and recommends non-parametric alternatives. Presents a practical guide covering the Wilcoxon signed-rank test (two classifiers, multiple datasets) and the Friedman + post-hoc Nemenyi test (multiple classifiers, multiple datasets).

**Directly required by the project:** the assignment explicitly asks for statistical hypothesis testing to determine if one classifier is significantly better than all others in the user-independent setting.

---

## Key Contributions

### The Problem with Paired t-Tests on Accuracy

Accuracy values are bounded in [0,1] and are not normally distributed — violating the t-test assumption. Moreover, comparing classifiers across many folds inflates type-I error. Demšar recommends non-parametric tests throughout.

---

## Recommended Tests

### 1. Comparing Two Classifiers — Wilcoxon Signed-Rank Test

**When to use:** Comparing exactly two classifiers across $k$ cross-validation folds (or $N$ datasets).

**Procedure:**
1. Compute per-fold accuracy differences $d_i = a_i^A - a_i^B$
2. Rank the absolute differences $|d_i|$ (ignoring ties where $d_i = 0$)
3. Compute $W^+ = \sum_{\{i: d_i > 0\}} \text{rank}(|d_i|)$ and $W^- = \sum_{\{i: d_i < 0\}} \text{rank}(|d_i|)$
4. Test statistic: $T = \min(W^+, W^-)$

Reject $H_0$ (classifiers are equivalent) if $T$ is below the critical value for $k$ and $\alpha = 0.05$.

For $k \geq 25$, use the normal approximation:
$$z = \frac{T - k(k+1)/4}{\sqrt{k(k+1)(2k+1)/24}}$$

---

### 2. Comparing Multiple Classifiers — Friedman Test + Nemenyi Post-Hoc

**When to use:** Comparing $k$ classifiers across $N$ datasets/folds.

**Step 1 — Friedman Test (omnibus):**

For each fold (row), rank the $k$ classifiers (1 = best, $k$ = worst). Let $r_i^j$ = rank of classifier $j$ in fold $i$, $R_j = \frac{1}{N}\sum_i r_i^j$ = average rank of classifier $j$.

$$\chi^2_F = \frac{12N}{k(k+1)}\left[\sum_j R_j^2 - \frac{k(k+1)^2}{4}\right]$$

Distributed as $\chi^2$ with $k-1$ degrees of freedom. If $p < 0.05$, at least one classifier is significantly different — proceed to post-hoc.

**Step 2 — Nemenyi Post-Hoc Test:**

The **Critical Difference (CD)** — if two classifiers' average ranks differ by more than CD, they are significantly different:
$$CD = q_\alpha \sqrt{\frac{k(k+1)}{6N}}$$

where $q_\alpha$ is from the studentised range distribution divided by $\sqrt{2}$ (tabulated values in the paper).

**Visualisation:** Plot a **Critical Difference diagram** — draw a horizontal axis of average ranks, mark each classifier, connect classifiers within CD of each other with a bar (non-significant differences).

---

### 3. McNemar's Test (Two Classifiers, Single Dataset)

For comparing two classifiers on a single fixed test set (standard pairwise comparison):

|  | B correct | B wrong |
|---|---|---|
| **A correct** | $n_{00}$ | $n_{01}$ |
| **A wrong** | $n_{10}$ | $n_{11}$ |

$$\chi^2 = \frac{(|n_{01} - n_{10}| - 1)^2}{n_{01} + n_{10}} \sim \chi^2_1$$

Reject $H_0$ at $\alpha = 0.05$ if $\chi^2 > 3.84$.

→ This is the most appropriate test for the **user-independent** setting where you have a fixed set of 10 users (folds).

---

## Which Test to Use for the Project?

The project has **10 cross-validation folds** (leave-one-user-out) and **4+ classifiers**. Demšar's recommendation:

| Comparison | $N$ folds | Test |
|---|---|---|
| 2 classifiers, 10 folds | $N=10$ | **Wilcoxon signed-rank** |
| All classifiers simultaneously | $N=10$, $k=4$ | **Friedman + Nemenyi** |
| 2 classifiers, 1 fixed test set | — | **McNemar's** |

**Practical code:**
```python
from scipy.stats import wilcoxon, friedmanchisquare

# Wilcoxon: compare DTW vs BiLSTM across 10 LOUO folds
stat, p = wilcoxon(dtw_accs, bilstm_accs)
print(f"Wilcoxon: stat={stat:.3f}, p={p:.4f}")

# Friedman: compare all 4 methods simultaneously
stat, p = friedmanchisquare(dtw_accs, edit_accs, dollar3_accs, bilstm_accs)
print(f"Friedman: chi2={stat:.3f}, p={p:.4f}")
```

---

## Connections

- Applied in → [[Workflows/Model_Evaluation|Model Evaluation]] (statistical tests section)
- Required by → [[Project - Gesture Recognition/Project Assignment|Project Assignment]] (user-independent statistical comparison)
- Alternative (single dataset) → McNemar's test → [[Workflows/Model_Evaluation|Model Evaluation]]
- Cross-validation protocol → [[Concepts/Cross_Validation|Cross-Validation Protocols]]
