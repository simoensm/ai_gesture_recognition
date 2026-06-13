---
title: "Papers — Map of Content"
tags:
  - MOC
  - literature
  - gesture-recognition
  - ai-foundations
created: 2026-05-20
updated: 2026-05-22
---

# Papers — Map of Content

> Central index of all literature notes. See also → [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Project MOC]]

---

## 🗃️ Dataset

| Paper | Year | Relevance |
|---|---|---|
| [[Huang2019]] — *Gesture-based system for next generation interfaces* | 2019 | **CRITICAL — our dataset** |
| [[Caputo2018_3DGestures]] — *Comparing 3D trajectories for simple mid-air gesture recognition* | 2018 | Benchmark comparison of DTW/$3/feature methods on mid-air gestures |

---

## ⏱️ Baseline: Dynamic Time Warping

| Paper | Year | What it contributes |
|---|---|---|
| [[Itakura1975]] — *Minimum prediction residual principle applied to speech recognition* | 1975 | Itakura parallelogram slope constraint for DTW |
| [[Sakoe1978]] — *DP algorithm for spoken word recognition* | 1978 | DTW algorithm + Sakoe-Chiba band |
| [[Myers1981_DTW]] — *Comparative study of DTW algorithms for connected-word recognition* | 1981 | Systematic comparison of step patterns, normalisation, slope constraints |
| [[Berndt1994]] — *Using DTW to find patterns in time series* | 1994 | DTW generalised to time series |
| [[Keogh2001_DDTW]] — *Derivative DTW* | 2001 | Shape-matching variant (DDTW) |
| [[Salvador2007]] — *FastDTW* | 2007 | $\mathcal{O}(n)$ approximation |
| [[Senin2008]] — *DTW Algorithm Review* | 2008 | Survey + implementation guide |
| [[Ding2008]] — *Experimental comparison of representations* | 2008 | 1-NN DTW is best-in-class baseline |

---

## ✏️ Baseline: Edit Distance

| Paper | Year | What it contributes |
|---|---|---|
| [[Levenshtein1966]] — *Binary codes for error correction* | 1966 | Edit distance definition |
| [[Wagner1974]] — *String-to-string correction problem* | 1974 | $\mathcal{O}(nm)$ DP algorithm |
| [[Nyirarugira2014]] — *Modified Levenshtein for gesture recognition* | 2014 | Gesture-specific adaptation, 93.9% acc |
| [[GarciaDiez2011_SumOverPaths]] — *Sum-over-paths extension of edit distances* | 2011 | Probabilistic generalisation over all alignments |
| [[Gavrila1999]] — *Visual analysis of human movement (survey)* | 1999 | Historical context, DTW vs edit distance |

---

## 💲 SOTA: $3 / Dollar Family

| Paper | Year | What it contributes |
|---|---|---|
| [[Wobbrock2007]] — *$1 Recognizer* | 2007 | 2D foundation: resample→rotate→scale→match |
| [[Kratz2010]] — *$3 Recognizer* | 2010 | 3D extension for accelerometer gestures |
| [[Kratz2011_Protractor3D]] — *Protractor3D* | 2011 | Closed-form rotation via Kabsch/SVD |
| [[Li2010_Protractor]] — *Protractor (2D)* | 2010 | Closed-form 2D rotation |
| [[Vatavu2012_P]] — *$P Recognizer* | 2012 | Unordered point-cloud variant |
| [[Vatavu2012_OneCent]] — *One-cent Recognizer* | 2012 | Centroid-distance rotation invariance |

---

## 🧠 SOTA: Deep Learning / BiLSTM

| Paper | Year | What it contributes |
|---|---|---|
| [[Hochreiter1997]] — *Long short-term memory* | 1997 | LSTM architecture + cell state |
| [[Gers2000]] — *Learning to forget* | 2000 | Forget gate for LSTM |
| [[Schuster1997]] — *Bidirectional RNNs* | 1997 | BiRNN architecture |
| [[Graves2005_BiLSTM]] — *BiLSTM for phoneme recognition* | 2005 | First BiLSTM > LSTM demonstration |
| [[Zhang2021_SkeletonBiLSTM]] — *3D skeletal gesture classification* | 2021 | BiLSTM on skeleton data (closest to our setup) |
| [[Zhu2019]] — *Smaller, faster skeleton recognition* | 2019 | Efficiency context |
| [[Ioffe2015_BatchNorm]] — *Batch Normalisation* | 2015 | Reduces internal covariate shift; LayerNorm preferred for BiLSTM |
| [[Srivastava2014_Dropout]] — *Dropout* | 2014 | Prevents co-adaptation; $p=0.3$ between LSTM layers |

---

## 🏛️ Seminal Foundations (Historical)

| Paper | Year | What it contributes |
|---|---|---|
| [[Rosenblatt1958_Perceptron]] — *The Perceptron* | 1958 | First trainable neural network model |
| [[Rumelhart1986_Backprop]] — *Learning representations by back-propagating errors* | 1986 | Backpropagation algorithm — makes MLPs trainable |
| [[LeCun1998_LeNet]] — *Gradient-Based Learning Applied to Document Recognition* | 1998 | LeNet-5 — first successful CNN, end-to-end learning |
| [[Krizhevsky2012_AlexNet]] — *ImageNet Classification with Deep CNNs (AlexNet)* | 2012 | Deep learning revolution — GPU training, ReLU, dropout |
| [[He2016_ResNet]] — *Deep Residual Learning for Image Recognition* | 2016 | Residual (skip) connections — trains 100+ layer networks |
| [[Vaswani2017_Transformer]] — *Attention Is All You Need* | 2017 | Transformer architecture — foundation of all LLMs |
| [[Devlin2018_BERT]] — *BERT: Pre-training of Deep Bidirectional Transformers* | 2018 | Pre-train → fine-tune paradigm, bidirectional LM |

---

## 🎲 Generative Models

| Paper | Year | What it contributes |
|---|---|---|
| [[Kingma2013_VAE]] — *Auto-Encoding Variational Bayes (VAE)* | 2013 | ELBO, reparameterisation trick, structured latent space |
| [[Goodfellow2014_GAN]] — *Generative Adversarial Nets* | 2014 | Adversarial training, minimax game, implicit generative models |

---

## ⚙️ Optimisation

| Paper | Year | What it contributes |
|---|---|---|
| [[Kingma2014_Adam]] — *Adam: A Method for Stochastic Optimization* | 2014 | Adaptive moment estimation — default DL optimizer |
| [[Ruder2016_Optimisers]] — *Overview of Gradient Descent Algorithms* | 2016 | Comprehensive survey of all GD variants |

---

## 🌲 Classical ML

| Paper | Year | What it contributes |
|---|---|---|
| [[Vapnik1995_SLT]] — *The Nature of Statistical Learning Theory* | 1995 | VC dimension, SRM, SVM theory |
| [[Cortes1995_SVM]] — *Support-Vector Networks* | 1995 | SVM, soft margin, kernel trick |
| [[Breiman2001_RandomForest]] — *Random Forests* | 2001 | Bagging + random features → robust ensemble |
| [[Friedman2001_GBM]] — *Greedy Function Approximation: A GBM* | 2001 | Gradient boosting as functional gradient descent |

---

## 📊 Supervised & Unsupervised ML

| Paper | Year | What it contributes |
|---|---|---|
| [[Tibshirani1996_Lasso]] — *Regression Shrinkage and Selection via the Lasso* | 1996 | L1 regularisation, sparsity, feature selection |
| [[Chen2020_SimCLR]] — *SimCLR: Contrastive Learning of Visual Representations* | 2020 | Self-supervised learning, NT-Xent contrastive loss |
| [[Demsar2006_StatisticalComparisons]] — *Statistical comparisons of classifiers over multiple data sets* | 2006 | Wilcoxon, Friedman + Nemenyi, McNemar — **required for assignment** |

---

## 📚 Textbooks & Surveys

| Reference | Key chapters for project |
|---|---|
| [[Bishop2006_PRML]] — *Pattern Recognition and Machine Learning* | Probabilistic ML bible — Bayesian, graphical models, HMMs |
| [[Hastie2009_ESL]] — *The Elements of Statistical Learning* | Statistical ML — regularisation, boosting, SVMs, unsupervised |
| [[Murphy2012_MLaPP]] — *Machine Learning: A Probabilistic Perspective* | Broadest coverage — every supervised/unsupervised algorithm |
| [[Goodfellow2016]] — *Deep Learning* (Goodfellow et al.) | Ch. 10: RNN/LSTM theory |
| [[Theodoridis2009]] — *Pattern Recognition, 4th ed.* | Edit distance, LCS, k-NN |
| [[Cover2006_InfoTheory]] — *Elements of Information Theory* | Entropy, KL divergence, mutual information |
| [[Deisenroth2020_MML]] — *Mathematics for Machine Learning* | Linear algebra, probability, optimisation, SVD, PCA |
| [[Fink2014_MarkovModels]] — *Markov Models for Pattern Recognition* | HMMs, Viterbi, Baum-Welch — sequence recognition |
| [[Rabiner1993_SpeechRecognition]] — *Fundamentals of Speech Recognition* | DTW slope constraints, HMM derivations |

---

## 🎬 Recommender Systems (MLSMM2156)

### Foundational CF

| Paper | Year | What it contributes |
|---|---|---|
| [[Resnick1994_GroupLens]] — *GroupLens* | 1994 | Coins "collaborative filtering"; first deployed CF system |
| [[Sarwar2001_ItemCF]] — *Item-based CF* | 2001 | Item-item similarity, adjusted cosine, scalable CF |
| [[Koren2009_MatrixFactorization]] — *MF for RecSys* | 2009 | SVD, bias terms, implicit feedback — Netflix Prize winner |
| [[Koren2008_FactorizationNeighborhood]] — *SVD++* | 2008 | Factorization + neighbourhood + implicit feedback |
| [[Lee1999_NMF]] — *NMF* | 1999 | Non-negative MF, parts-based representations |

### Dataset & Features

| Paper | Year | What it contributes |
|---|---|---|
| [[Harper2016_MovieLens]] — *MovieLens Datasets* | 2016 | Standard benchmark; 1M dataset used in project |
| [[Vig2012_TagGenome]] — *Tag Genome* | 2012 | 1,128 continuous tag scores — best content feature in project |

### Content-Based & Hybrid

| Paper | Year | What it contributes |
|---|---|---|
| [[Burke2002_HybridRS]] — *Hybrid RS Survey* | 2002 | 7 hybridisation strategies; cascade pattern used in webapp |
| [[Schein2002_ColdStart]] — *Cold-Start Recommendations* | 2002 | Methods for new users/items; justifies Phase 0/1 cascade |

### Évaluation des RecSys

| Paper | Year | What it contributes |
|---|---|---|
| [[Herlocker2004_EvaluatingCF]] — *Evaluating CF Systems* | 2004 | RMSE vs MAE, coverage, novelty, offline vs user study |
| [[Chen2017_PerformanceEvaluation]] — *Performance Evaluation of RS* | 2017 | Cadre complet : 3 méthodes (offline/user study/A-B) × 4 perspectives métriques (ML, IR, HCI, Eng.) |
| [[Castells2015_NoveltyDiversity]] — *Novelty and Diversity in RS* | 2015 | ILD, MIUF, Unexpectedness, Aggregate Diversity — au-delà de la précision ; MMR greedy algorithm |
| [[Meng2020_DataSplitting]] — *Exploring Data Splitting Strategies* | 2020 | Leave One Last vs Temporal Global — la stratégie de découpage est une variable confondante critique |

---

## 🔗 Concept Notes

| Concept | Used by |
|---|---|
| [[Concepts/Supervised_Learning_Framework\|Supervised Learning Framework]] | All supervised methods |
| [[Concepts/Unsupervised_Learning\|Unsupervised Learning]] | Clustering, density estimation, representations |
| [[Concepts/Linear_Regression\|Linear Regression]] | OLS, ridge, lasso |
| [[Concepts/Logistic_Regression\|Logistic Regression]] | Classification, softmax, cross-entropy |
| [[Concepts/Probabilistic_Classifiers\|Probabilistic Classifiers]] | Naive Bayes, LDA, QDA |
| [[Concepts/Semi_Supervised_Self_Supervised\|Semi/Self-Supervised Learning]] | Contrastive, SimCLR, MAE, BYOL |
| [[Concepts/Dynamic_Programming\|Dynamic Programming]] | DTW, Edit Distance, LCS |
| [[Concepts/Vector_Quantization\|Vector Quantization]] | Edit Distance |
| [[Concepts/kNN_Classifier\|k-NN Classifier]] | All three baselines |
| [[Concepts/Cross_Validation\|Cross-Validation Protocol]] | All methods |
| [[Concepts/Kabsch_Algorithm\|Kabsch Algorithm]] | $3 Recognizer |
| [[Concepts/Recommender_Systems\|Recommender Systems]] | MLSMM2156 project — overview & taxonomy |
| [[Concepts/Collaborative_Filtering\|Collaborative Filtering]] | UserBased, ItemBased CF models |
| [[Concepts/Matrix_Factorization\|Matrix Factorization]] | SVD, SVD++, NMF models |
| [[Concepts/Content_Based_Filtering\|Content-Based Filtering]] | ContentBased, ContentBasedWithStats |
| [[Concepts/TF-IDF\|TF-IDF]] | Text feature extraction; genome tags |

---

## 🔄 Import New Papers from Zotero

1. Add paper to Zotero (File → Import if you have a .bib entry)
2. In Obsidian: `Ctrl+P` → **Zotero Integration: Import Notes** → select **Literature Note (Gesture Recognition)**
3. Paper note is automatically created in `Papers/{{citekey}}.md`
4. Fill in **Relevance** and **Connections** sections manually
5. Add `[[citekey]]` links in the relevant method notes
