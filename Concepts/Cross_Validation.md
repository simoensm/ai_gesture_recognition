---
title: "Cross-Validation Protocol (Project-Specific)"
tags:
  - concept
  - cross-validation
  - evaluation
  - user-independent
  - user-dependent
created: 2026-05-20
---

# Cross-Validation Protocol (Project-Specific)

> [!abstract] Context
> This note describes the exact cross-validation protocols required by the [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]] guidelines. Two settings are required: **user-independent** and **user-dependent**.

---

## Dataset Structure (from [[Papers/Huang2019]])

- 10 users × 10 gesture classes × 10 repetitions = **1,000 sequences**
- Variable-length sequences, 3D coordinates

---

## Setting 1: User-Independent (Leave-One-User-Out)

**Goal:** Measure how well the model generalises to an **unseen user**.

**Protocol:**
1. Hold out all 100 sequences from user $u$ (10 classes × 10 repetitions)
2. Train on remaining 900 sequences (9 users × 10 × 10)
3. Evaluate on user $u$'s sequences
4. Repeat for all 10 users

**Result:** 10 accuracy values → report **mean ± std**

**Statistical test required:** Hypothesis testing to evaluate if the best model is significantly better than all others.

```
Users:    1   2   3   4   5   6   7   8   9   10
Fold 1:  [test] [   train   ================================]
Fold 2:  [tr] [test] [   train   =========================]
...
Fold 10: [   train   ================================] [test]
```

---

## Setting 2: User-Dependent (Leave-One-Sample-Out)

**Goal:** Measure how well the model recognises gestures from a **known user**.

**Protocol:**
1. For each user $u$ separately:
   - Hold out one repetition of each gesture class (10 samples)
   - Train on remaining 9 repetitions (90 samples for this user)
   - Evaluate on the 10 held-out samples
2. Repeat 10 times (once per repetition index)
3. **Also repeat across all 10 users**

**Result:** 10 × 10 = 100 accuracy values → report **mean ± std** (or 10 per user)

---

## Expected Results

> [!info] Guideline expectation
> User-dependent accuracy > User-independent accuracy
>
> This is because user-dependent training includes examples from the same user, capturing their personal gesture style. User-independent must generalise across completely new users.

---

## Implementation Notes

- **Fix random seeds** and save train/test splits before running any model — all methods must use the same splits for fair comparison
- **Codebook leakage:** For edit distance, fit the k-means codebook only on the training fold (per the guidelines, fitting once on the full dataset is allowed for simplicity)
- **Statistical test (user-independent only):** Use a paired test (e.g., McNemar's test or Wilcoxon signed-rank) across fold accuracies to determine if the best model is significantly better

---

## Connections

- Used in → [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|DTW]], [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance]], [[Project - Gesture Recognition/03_Three_Cent_Recognizer|$3]], [[Project - Gesture Recognition/04_BiLSTM|BiLSTM]]
- Dataset → [[Papers/Huang2019]]
- Concept → [[kNN_Classifier|k-NN Classifier]]
