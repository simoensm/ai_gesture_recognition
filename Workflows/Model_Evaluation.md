---
title: "Model Evaluation & Performance Metrics"
tags: [workflow, evaluation, metrics, confusion-matrix, cross-validation, statistical-tests, practical]
created: 2026-05-22
---

# Model Evaluation & Performance Metrics

> How to rigorously measure, report, and compare model performance. Choosing the right metric matters as much as choosing the right model.

---

## Classification Metrics

### Confusion Matrix

For $K$ classes, the confusion matrix $C \in \mathbb{R}^{K \times K}$:
$$C_{ij} = \text{number of samples with true class } i \text{ predicted as class } j$$

Diagonal entries = correct predictions; off-diagonal = misclassifications.

### Derived Metrics (Binary)

| Metric | Formula | Intuition |
|---|---|---|
| **Accuracy** | $(TP+TN)/(P+N)$ | Overall correct fraction |
| **Precision** | $TP/(TP+FP)$ | Of predicted positives, how many are real? |
| **Recall (Sensitivity)** | $TP/(TP+FN)$ | Of real positives, how many did we find? |
| **Specificity** | $TN/(TN+FP)$ | Of real negatives, how many did we reject? |
| **F1-score** | $2 \cdot P \cdot R / (P+R)$ | Harmonic mean of precision and recall |
| **AUC-ROC** | Area under ROC curve | Ranking quality across all thresholds |

### Multiclass Extensions

- **Macro-average:** Compute metric per class, then average (treats all classes equally regardless of size)
- **Micro-average:** Pool all TP/FP/FN across classes before computing (dominated by frequent classes)
- **Weighted-average:** Weight by class support

For the gesture project (balanced 10-class): macro accuracy = micro accuracy = overall accuracy.

---

## Regression Metrics

| Metric | Formula | Notes |
|---|---|---|
| **MSE** | $\frac{1}{n}\sum(y-\hat{y})^2$ | Sensitive to outliers (quadratic) |
| **MAE** | $\frac{1}{n}\sum|y-\hat{y}|$ | Robust to outliers |
| **RMSE** | $\sqrt{MSE}$ | Same units as $y$ |
| **$R^2$** | $1 - SS_{res}/SS_{tot}$ | Fraction of variance explained |
| **MAPE** | $\frac{100}{n}\sum|(y-\hat{y})/y|$ | Relative error; undefined for $y=0$ |

---

## Learning Curves

Plot training loss and validation loss vs. training epochs (or dataset size):

```
Loss
 │  ╲ train loss
 │   ╲___________  ← underfitting: both losses high
 │           ╲
 │    val loss ╲_______ ← overfitting: gap between train & val
 │
 └─────────────────────── Epochs
```

**Diagnosis:**
- High train AND val loss → underfitting (model too simple, insufficient training)
- Low train loss, high val loss → overfitting (increase regularisation, more data, smaller model)
- Both converging nicely → good generalisation

---

## Cross-Validation Reporting

When using $k$-fold CV, always report:
- **Mean accuracy:** $\bar{a} = \frac{1}{k}\sum_{i=1}^k a_i$
- **Standard deviation:** $\sigma_a = \sqrt{\frac{1}{k-1}\sum(a_i-\bar{a})^2}$
- **Format:** $\bar{a} \pm \sigma_a$ (e.g., $91.3\% \pm 2.1\%$)

The standard deviation reflects both model variance and fold-to-fold data variability.

---

## Statistical Significance Testing

> **Key reference:** [[Papers/Demsar2006_StatisticalComparisons|Demšar (2006)]] — the standard guide for comparing classifiers across multiple datasets/folds. **Required citation for the assignment statistical tests section.**

### McNemar's Test (Comparing Two Classifiers)

For two classifiers A and B on the same test set:

|  | B correct | B wrong |
|---|---|---|
| **A correct** | $n_{00}$ | $n_{01}$ |
| **A wrong** | $n_{10}$ | $n_{11}$ |

$$\chi^2 = \frac{(|n_{01} - n_{10}| - 1)^2}{n_{01} + n_{10}} \sim \chi^2_1$$

Reject $H_0$ (classifiers perform equally) at $p < 0.05$ if $\chi^2 > 3.84$.

**Use this when comparing two models on the same fixed test set.**

### Paired t-test (CV Results)

When comparing models via $k$-fold CV, the per-fold accuracy differences $d_i = a_i^A - a_i^B$ follow a t-distribution:
$$t = \frac{\bar{d}}{s_d / \sqrt{k}} \sim t_{k-1}$$

### 5×2-CV Test (Dietterich, 1998)

More robust alternative — repeat 2-fold CV 5 times with different random splits. Avoids inflated degrees of freedom from standard repeated CV.

---

## Calibration

A **well-calibrated** model's predicted probability $p$ reflects actual frequency — among all samples where the model predicts 80% confidence, ~80% should be correct.

- **Reliability diagram (calibration plot):** Bin predictions; plot mean confidence vs. mean accuracy per bin
- **Expected Calibration Error (ECE):** Weighted mean of |confidence − accuracy| across bins
- **Temperature scaling:** Post-hoc calibration by dividing logits by temperature $T > 1$ (softens over-confident models)

---

## Error Analysis Protocol

After evaluating, investigate failures systematically:

1. **Confusion matrix heatmap:** Identify which class pairs are most confused
2. **Look at individual failures:** Are they ambiguous gestures? Bad recordings? Labelling errors?
3. **Stratify by metadata:** Error rate by user, by sequence length, by recording session
4. **Systematic vs. random:** If a specific class always fails, it's a systematic bias in the model or data

---

## Connections

- Cross-validation setup → [[Concepts/Cross_Validation|Cross-Validation Protocols]]
- Bias-variance trade-off → [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]]
- Full pipeline → [[Workflows/ML_Pipeline|ML Pipeline]]
- Hyperparameter tuning → [[Workflows/Hyperparameter_Tuning|Hyperparameter Tuning]]
- Statistical learning theory → [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]]
