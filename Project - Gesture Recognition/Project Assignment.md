---
title: MLSM2154 — Gesture Recognition Project Assignment
course: MLSM2154
tags:
  - project
  - gesture-recognition
  - supervised-learning
  - sequence-classification
  - assignment
  - deep-learning
  - baseline
  - cross-validation
created: 2026-05-22
deadline: 2026-05-24
authors:
  - Saerens, Marco
---

# Objective

The objective of this project is to put into practice some of the techniques introduced in the [[Concepts/Supervised_Learning_Framework|statistical machine-learning]] lectures. This is done through the study of a practical case which requires the use of a scientific programming language, as well as libraries, related to the statistical processing of data (Python in this case). More precisely, this work aims at applying data science/[[Concepts/Supervised_Learning_Framework|statistical learning]] techniques for [[Concepts/Supervised_Learning_Framework|supervised classification]] of three-dimensional recorded hand gestures. This project will be carried out by groups of maximum 3 students – the same groups as for the other courses of the major.

---

# Problem Statement

You are a young data scientist hired by an IT company. Your first work is to implement a hand gesture recognition system for a smart user interface. A three-dimensional tracking of the position vector $r(t) = [x(t), y(t), z(t)]^\top$ of the hand is recorded as a sequence of the three coordinates in function of the (discrete) time $t$. Thus, the user is executing a sketch or a symbol with his hand, which is recorded and then recognized by the system. Sketches and symbols are rapidly executed freehand drawings ([[Papers/Huang2019|Huang et al., 2019]]). Here, for the first dataset, the sketches are representations of the numbers from 0 to 9 (10 classes or categories). The second dataset contains three-dimensional (3D) figures. You are asked to develop a sketch recognition system identifying the category of the sequence, and implementing/testing it in the Python programming language.

---

# Datasets

In order to train the recognition system, some publicly available gestures dataset, gathered by and described in [[Papers/Huang2019|Huang et al. (2019)]], are used. Concerning the first dataset studied by the authors of this paper, ten users/subjects participated and were each asked to draw ten times the numbers {0, 1, 2, 3, 4, 5, 6, 7, 8, 9} (called "domain 1" data in [[Papers/Huang2019|Huang et al. (2019)]]) in 3D. We therefore have ten repetitions of each of these ten numbers by ten subjects, that is, **1000 sequences** of 3D coordinates in total. You will also have to analyse the fourth dataset (called "domain 4" data in [[Papers/Huang2019|Huang et al. (2019)]]) representing 3D figures with the same number of subjects, figures, and repetitions.

> **Structure:** 10 users × 10 classes × 10 repetitions = 1,000 sequences per domain.  
> **Features:** 3D coordinates $(x_t, y_t, z_t)$ at each time step $t = 1, 2, 3, \ldots$

---

# Implementation of the Models

The project is very open. Notice that you will first have to perform a preliminary **exploratory analysis** of the data as well as some [[Workflows/Data_Preprocessing|pre-processing steps]] (like, for instance, [[Workflows/Data_Preprocessing|standardise the signals]], [[Concepts/Dimensionality_Reduction|PCA]], etc) in order to facilitate the classification. Note also that we will not use the precise recorded time $t$ of the 3D observations; we consider that the time steps are constant ($t = 1, 2, 3, \ldots$) even if this hypothesis is not completely true.

---

# Baseline Methods

You should then implement a baseline technique like the [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|dynamic time warping]] (see the slides of the course and, for instance, [[Papers/Rabiner1993_SpeechRecognition|Rabiner & Juang (1993)]]), the [[Project - Gesture Recognition/02_Edit_Distance|edit-distance]] or, alternatively, the [[Concepts/Longest_Common_Subsequence|longest common subsequence]] ([[Papers/Theodoridis2009|Theodoridis & Koutroumbas, 2009]]) after having performed a [[Concepts/Vector_Quantization|vector quantization (clustering)]] of the signals for the last two methods. Then, you simply rely on a [[Concepts/kNN_Classifier|k nearest-neighbor]] technique.

You **must implement yourself** at least two of these three baselines (including necessarily [[Concepts/Dynamic_Programming|dynamic programming]]) in Python 3 and thus obtain baseline results on the dataset (you are not allowed to use a library implementing these algorithms for this part, but you are free to use libraries facilitating scientific computation like numpy, etc).

[[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping|Dynamic time warping]] can work directly on the 3D coordinates of the signal, but the [[Project - Gesture Recognition/02_Edit_Distance|edit distance]] and the [[Concepts/Longest_Common_Subsequence|longest common subsequence]] are designed in order to compare strings (sequences of characters or symbols). Therefore, to be able to compute the edit-distance/longest common subsequence, you need to convert your signals into a string. This can be done, for instance, using a [[Concepts/Clustering_EM|clustering technique]]. More precisely, this is done by:

1. Gathering every 3D coordinate of every recorded gesture of the training set into one big set of 3D coordinates
2. Applying [[Concepts/Clustering_EM|k-means]] and recording the coordinates of the centroid of each cluster
3. For each gesture, replacing each spatial 3D point with the label of the closest centroid

This is [[Concepts/Vector_Quantization|vector quantization]] — instead of a series of 3D coordinates, you now have a **sequence of labels** representing visited clusters along time (e.g. `AAACCCBAA...`). You can then use these sequences to compare gestures using [[Project - Gesture Recognition/02_Edit_Distance|edit distance]] or [[Concepts/Longest_Common_Subsequence|LCS]].

> **Note on rigor:** The most rigorous method would require performing the [[Concepts/Clustering_EM|clustering]] inside the [[Concepts/Cross_Validation|cross-validation]] only on the training set. For simplicity, you are allowed to cluster once before performing cross-validation.

---

# More State-of-the-Art Methods

Thereafter, you are free to use and test at least two other, more state-of-the-art, methods found in the literature, like the [[Project - Gesture Recognition/03_Three_Cent_Recognizer|$1 recognizer]] (see the references below), [[Project - Gesture Recognition/04_BiLSTM|recurrent neural networks]], [[Concepts/Neural_Networks_MLP|deep learning techniques]] ([[Papers/Goodfellow2016|Goodfellow et al., 2016]]), or any other technique for classifying sequential data you are interested in. For this second part, you are allowed to use existing implementations/libraries in Python and, of course, refer to them in the report. The idea is to **compare the results** obtained by this new technique(s) to the baseline techniques. One simple possibility would be to [[Workflows/Data_Preprocessing|extract meaningful features]] from the 3D signals and then rely on a simple [[Concepts/Supervised_Learning_Framework|supervised classification model]].

---

# Validation and Comparison Tests

You will evaluate and compare the [[Workflows/Model_Evaluation|classification accuracy]] of your models by [[Concepts/Cross_Validation|cross-validation]] on two different task settings: **user-independent** and **user-dependent** gesture identification.

## User-Independent Setting

Here, for the [[Concepts/Cross_Validation|cross-validation]], you have to use a **leave-one-user-out** procedure:
- Use 90% of the data from 9 users for training, test on the remaining user
- Repeat 10 times (hold each user out in turn)

This quantifies the ability of identifying the gestures of a **new user** not present in the training set. For each method, record the different accuracies across users and compute the **mean accuracy and standard deviation** → see [[Workflows/Model_Evaluation|Model Evaluation]].

## User-Dependent Setting

Use a **leave-one-gesture-sample-out** procedure:
- For each user, hold out one example of each of the 10 gestures as test fold
- Repeat ten times per user

For each user, 9 realisations of each gesture are used for training; performance is evaluated on folds containing one sample per class per user. We expect results to be more accurate in the user-dependent setting.

## Statistical Comparison

For the **user-independent** case, perform **[[Workflows/Model_Evaluation|statistical hypothesis tests]]** to evaluate whether one classification model is **significantly better** than all others. Provide:

- A **results table**: mean ± std per method × {user-dependent, user-independent} → [[Workflows/Model_Evaluation|Model Evaluation]]
- A **[[Workflows/Model_Evaluation|confusion matrix]]** showing which gesture classes are most confused by the best model
- **[[Workflows/Model_Evaluation|Statistical tests]]** (e.g. McNemar's test) comparing models on the user-independent setting

---

# Report

Please do not forget to mention your affiliation on the cover page, together with your names and group number. Write a report **(PDF) in English of maximum 10 pages** (everything included; additional content may go in an appendix). The report must have a **professional, scientific look** — integrate plots, equations (no screenshots of equations/code), and references.

**Required sections:**

- [ ] Short **introduction** (problem, dataset, methods overview)
- [ ] **Theoretical explanation** with main equations of all investigated methods — especially the advanced ones
- [ ] Short **description of implementation and methodology**
- [ ] **Description and discussion** of comparison results
- [ ] **Conclusion**

**Evaluation criteria:**

- Quality of report and code
- Amount of experimental work (including potential additional experiments)
- Relevance and description of the investigated methods

> Do not focus on the code itself in the report. A ReadMe file in the zip may provide code details.

**Deadline:** May 24, 2026, 23:59 on Moodle (section "Project"). Penalty: −0.5/10 per day late; maximum delay 3 days.

---

# Methods Notes

| Method | Note | Role |
|---|---|---|
| Dynamic Time Warping | [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping\|01 — DTW]] | Baseline |
| Edit Distance | [[Project - Gesture Recognition/02_Edit_Distance\|02 — Edit Distance]] | Baseline |
| $3 Recognizer | [[Project - Gesture Recognition/03_Three_Cent_Recognizer\|03 — $3 Recognizer]] | SOTA template |
| BiLSTM | [[Project - Gesture Recognition/04_BiLSTM\|04 — BiLSTM]] | SOTA deep learning |

---

# Key References

| Paper | Role |
|---|---|
| [[Papers/Huang2019\|Huang et al. (2019)]] | Dataset — **critical** |
| [[Papers/Rabiner1993_SpeechRecognition\|Rabiner & Juang (1993)]] | DTW foundations and slope constraints |
| [[Papers/Theodoridis2009\|Theodoridis & Koutroumbas (2009)]] | Edit distance, LCS, pattern recognition |
| [[Papers/GarciaDiez2011_SumOverPaths\|Garcia-Diez et al. (2011)]] | Sum-over-paths edit distance extension |
| [[Papers/Wobbrock2007\|Wobbrock et al. (2007)]] | $1 Recognizer |
| [[Papers/Kratz2010\|Kratz & Rohs (2010)]] | $3 Recognizer (3D extension) |
| [[Papers/Hochreiter1997\|Hochreiter & Schmidhuber (1997)]] | LSTM |
| [[Papers/Goodfellow2016\|Goodfellow et al. (2016)]] | Deep learning textbook |

---

# Concept Links

| Concept | Relevance |
|---|---|
| [[Concepts/Supervised_Learning_Framework\|Supervised Learning Framework]] | Overall framing — ERM, loss, generalisation |
| [[Concepts/Dynamic_Programming\|Dynamic Programming]] | Shared backbone of DTW, edit distance, LCS |
| [[Concepts/Longest_Common_Subsequence\|Longest Common Subsequence (LCS)]] | Baseline alternative to edit distance |
| [[Concepts/Vector_Quantization\|Vector Quantization]] | Converting 3D sequences to symbol strings |
| [[Concepts/Clustering_EM\|Clustering & k-Means]] | Building the VQ codebook |
| [[Concepts/kNN_Classifier\|k-NN Classifier]] | Decision rule for all baseline methods |
| [[Concepts/Cross_Validation\|Cross-Validation Protocols]] | Leave-one-user-out and leave-one-sample-out |
| [[Concepts/Dimensionality_Reduction\|PCA / Dimensionality Reduction]] | Optional preprocessing step |
| [[Workflows/Data_Preprocessing\|Data Preprocessing]] | Standardisation, resampling, feature engineering |
| [[Workflows/Model_Evaluation\|Model Evaluation]] | Accuracy, confusion matrix, statistical tests |
| [[Workflows/ML_Pipeline\|ML Pipeline]] | End-to-end workflow reference |
