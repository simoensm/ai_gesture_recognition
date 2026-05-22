---
title: "AI Knowledge Graph — Master Map of Content"
tags: [MOC, ai, machine-learning, knowledge-graph, master]
created: 2026-05-22
---

# AI Knowledge Graph — Master Map of Content

> The top-level index of this Obsidian vault. Navigate by clicking any link. Everything flows from mathematical foundations up through concepts, literature, and applied workflows to your project.

---

## 🗺️ Navigation by Layer

```
┌─────────────────────────────────────────────────────┐
│              Your Project (Applied)                  │
│  [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|🤚 Gesture Recognition Project]]   │
└──────────────────────┬──────────────────────────────┘
                       │ implements / evaluates
┌──────────────────────▼──────────────────────────────┐
│                  Workflows                           │
│  ML Pipeline · Preprocessing · Evaluation · Tuning  │
└──────────────────────┬──────────────────────────────┘
                       │ applies / references
┌──────────────────────▼──────────────────────────────┐
│               Literature (Papers)                    │
│  Seminal papers · Survey papers · Textbooks          │
└──────────────────────┬──────────────────────────────┘
                       │ explains / implements
┌──────────────────────▼──────────────────────────────┐
│              Concepts & Algorithms                   │
│  Neural networks · Attention · SVM · Ensembles …     │
└──────────────────────┬──────────────────────────────┘
                       │ built on
┌──────────────────────▼──────────────────────────────┐
│           Mathematical Foundations                   │
│  Probability · Optimisation · Information Theory …   │
└─────────────────────────────────────────────────────┘
```

---

## 🔢 Mathematical Foundations

The bedrock that all of ML rests on. Start here if you need to understand *why* an algorithm works.

| Note | Core ideas |
|---|---|
| [[Foundations/Probability_Theory\|Probability Theory]] | Sample spaces, distributions, Bayes' theorem, CLT, inequalities |
| [[Foundations/Bayesian_Inference\|Bayesian Inference]] | Prior, posterior, MAP, conjugate priors, variational inference, ELBO |
| [[Foundations/Maximum_Likelihood_Estimation\|Maximum Likelihood Estimation]] | MLE → loss functions, MAP → regularisation, Fisher information |
| [[Foundations/Information_Theory\|Information Theory]] | Entropy, KL divergence, cross-entropy loss, mutual information, JSD |
| [[Foundations/Linear_Algebra_for_ML\|Linear Algebra for ML]] | Norms, eigendecomposition, SVD, PCA, Mahalanobis distance |
| [[Foundations/Statistical_Learning_Theory\|Statistical Learning Theory]] | VC dimension, PAC learning, bias-variance decomposition, SRM |
| [[Foundations/Optimisation_Theory\|Optimisation Theory]] | GD, SGD, Adam, momentum, learning rate scheduling, gradient clipping |

→ See [[Foundations/00_Foundations_MOC|Foundations MOC]] for the full index.

---

## 🧠 Concepts & Algorithms

Core ML algorithms and architectural concepts. These sit between theory and specific papers.

### Supervised Learning
| Note | Core ideas |
|---|---|
| [[Concepts/Supervised_Learning_Framework\|Supervised Learning Framework]] | ERM, loss functions, generalisation, bias-variance, test set protocol |
| [[Concepts/Linear_Regression\|Linear Regression]] | OLS, ridge, lasso, probabilistic interpretation, polynomial features |
| [[Concepts/Logistic_Regression\|Logistic Regression]] | Sigmoid, softmax, cross-entropy, discriminative classification |
| [[Concepts/Probabilistic_Classifiers\|Probabilistic Classifiers]] | Naive Bayes, LDA, QDA, generative vs. discriminative |
| [[Concepts/Bias_Variance_Regularisation\|Bias-Variance & Regularisation]] | L1/L2/dropout/early stopping, the fundamental trade-off |
| [[Concepts/Kernel_Methods_SVM\|Kernel Methods & SVM]] | Margin maximisation, kernel trick, Mercer's theorem, soft margin |
| [[Concepts/Decision_Trees_Ensembles\|Decision Trees & Ensembles]] | ID3/CART, random forest (bagging), gradient boosting, XGBoost |

### Unsupervised & Self-Supervised Learning
| Note | Core ideas |
|---|---|
| [[Concepts/Unsupervised_Learning\|Unsupervised Learning]] | Taxonomy: clustering, density estimation, representation learning |
| [[Concepts/Clustering_EM\|Clustering & EM Algorithm]] | k-means, GMM, EM, DBSCAN, hierarchical, Baum-Welch |
| [[Concepts/Dimensionality_Reduction\|Dimensionality Reduction]] | PCA, t-SNE, UMAP, autoencoders, bottleneck representations |
| [[Concepts/Generative_Models\|Generative Models]] | VAE (ELBO), GAN (minimax), Diffusion models, score matching |
| [[Concepts/Semi_Supervised_Self_Supervised\|Semi/Self-Supervised Learning]] | Contrastive learning, SimCLR, BYOL, MAE, pseudo-labelling |

### Neural Networks & Deep Learning
| Note | Core ideas |
|---|---|
| [[Concepts/Backpropagation\|Backpropagation]] | Chain rule, vanishing/exploding gradients, autograd frameworks |
| [[Concepts/Neural_Networks_MLP\|Neural Networks & MLP]] | Activation functions, universal approximation, initialisation |
| [[Concepts/Convolutional_Neural_Networks\|Convolutional Neural Networks]] | Convolution, pooling, ResNet, transfer learning |
| [[Concepts/Attention_Transformer\|Attention & Transformers]] | Scaled dot-product, multi-head, positional encoding, BERT vs. GPT |

### Gesture Recognition Specific
| Note | Core ideas |
|---|---|
| [[Concepts/Dynamic_Programming\|Dynamic Programming]] | Optimal substructure; DTW, edit distance, LCS as DP instances |
| [[Concepts/Vector_Quantization\|Vector Quantization]] | k-means codebook for edit distance alphabet |
| [[Concepts/kNN_Classifier\|k-NN Classifier]] | 1-NN decision rule, distance sensitivity, curse of dimensionality |
| [[Concepts/Cross_Validation\|Cross-Validation Protocols]] | User-independent (LOUO), user-dependent (LOSO) |
| [[Concepts/Kabsch_Algorithm\|Kabsch Algorithm]] | SVD-based optimal rotation alignment ($3 Recognizer) |

---

## 📄 Literature (Papers)

### Historical Foundations
| Paper | Year | Contribution |
|---|---|---|
| [[Papers/Rosenblatt1958_Perceptron\|Rosenblatt 1958]] | 1958 | The Perceptron |
| [[Papers/Rumelhart1986_Backprop\|Rumelhart 1986]] | 1986 | Backpropagation |
| [[Papers/LeCun1998_LeNet\|LeCun 1998]] | 1998 | LeNet-5, CNNs |
| [[Papers/Hochreiter1997\|Hochreiter 1997]] | 1997 | LSTM |
| [[Papers/Krizhevsky2012_AlexNet\|Krizhevsky 2012]] | 2012 | AlexNet — deep learning revolution |
| [[Papers/He2016_ResNet\|He 2016]] | 2016 | ResNet — residual connections |
| [[Papers/Vaswani2017_Transformer\|Vaswani 2017]] | 2017 | Transformer — attention is all you need |
| [[Papers/Devlin2018_BERT\|Devlin 2018]] | 2018 | BERT — pre-train then fine-tune |

### Generative Models
| Paper | Year | Contribution |
|---|---|---|
| [[Papers/Kingma2013_VAE\|Kingma 2013]] | 2013 | VAE — reparameterisation trick, ELBO |
| [[Papers/Goodfellow2014_GAN\|Goodfellow 2014]] | 2014 | GAN — adversarial training |

### Optimisation
| Paper | Year | Contribution |
|---|---|---|
| [[Papers/Kingma2014_Adam\|Kingma 2014]] | 2014 | Adam optimiser |
| [[Papers/Ruder2016_Optimisers\|Ruder 2016]] | 2016 | GD algorithms overview |

### Classical ML Theory
| Paper | Year | Contribution |
|---|---|---|
| [[Papers/Vapnik1995_SLT\|Vapnik 1995]] | 1995 | Statistical Learning Theory, VC dimension |
| [[Papers/Cortes1995_SVM\|Cortes 1995]] | 1995 | Support Vector Machines |
| [[Papers/Tibshirani1996_Lasso\|Tibshirani 1996]] | 1996 | Lasso — L1 regularisation, sparsity |
| [[Papers/Breiman2001_RandomForest\|Breiman 2001]] | 2001 | Random Forests |
| [[Papers/Friedman2001_GBM\|Friedman 2001]] | 2001 | Gradient Boosting Machines |

### Self-Supervised Learning
| Paper | Year | Contribution |
|---|---|---|
| [[Papers/Chen2020_SimCLR\|Chen 2020]] | 2020 | SimCLR — contrastive self-supervised visual representations |

### Textbooks (Essential)
| Textbook | Coverage |
|---|---|
| [[Papers/Bishop2006_PRML\|Bishop 2006 (PRML)]] | Probabilistic ML — Bayesian, graphical models, HMMs, variational inference |
| [[Papers/Hastie2009_ESL\|Hastie 2009 (ESL)]] | Statistical ML — regularisation, boosting, SVMs, unsupervised |
| [[Papers/Murphy2012_MLaPP\|Murphy 2012 (MLaPP)]] | Broadest coverage — every supervised/unsupervised algorithm, probabilistic |
| [[Papers/Deisenroth2020_MML\|Deisenroth 2020 (MML)]] | Math foundations — linear algebra, calculus, probability, optimisation |

### Gesture Recognition Papers
See → [[Papers/00_Papers_MOC|Papers MOC]] for the complete list (DTW, edit distance, $3 family, BiLSTM).

---

## ⚙️ Workflows

Practical step-by-step guides for applying ML.

| Workflow | What it covers |
|---|---|
| [[Workflows/ML_Pipeline\|End-to-End ML Pipeline]] | Full pipeline overview, 8 stages from raw data to deployment |
| [[Workflows/Data_Preprocessing\|Data Preprocessing]] | Cleaning, normalisation, resampling, feature engineering, augmentation |
| [[Workflows/Model_Evaluation\|Model Evaluation]] | Metrics, confusion matrix, learning curves, statistical tests, calibration |
| [[Workflows/Hyperparameter_Tuning\|Hyperparameter Tuning]] | Grid search, random search, Bayesian optimisation, learning rate finder |

---

## 🤚 Your Project

| Note | Status |
|---|---|
| [[Project - Gesture Recognition/00_Gesture_Recognition_MOC\|Gesture Recognition MOC]] | Master index, checklist |
| [[Project - Gesture Recognition/01_DTW_Dynamic_Time_Warping\|01 — DTW]] | Sakoe-Chiba, DDTW, FastDTW |
| [[Project - Gesture Recognition/02_Edit_Distance\|02 — Edit Distance]] | Levenshtein, LCS, Nyirarugira adaptation |
| [[Project - Gesture Recognition/03_Three_Cent_Recognizer\|03 — $3 Recognizer]] | Resampling pipeline, Protractor3D, Kabsch |
| [[Project - Gesture Recognition/04_BiLSTM\|04 — BiLSTM]] | Architecture, training, comparison |

**Dataset:** Huang et al. (2019) — 10 users × 10 classes × 10 repetitions = 1,000 sequences, 3D coordinates. → [[Papers/Huang2019|Huang 2019]]

---

## 🔍 Quick Reference: "What do I look up for…"

| Question | Go to |
|---|---|
| Why does gradient descent work? | [[Foundations/Optimisation_Theory\|Optimisation Theory]] |
| How is KL divergence defined? | [[Foundations/Information_Theory\|Information Theory]] |
| How does backprop compute gradients? | [[Concepts/Backpropagation\|Backpropagation]] |
| What is VC dimension? | [[Foundations/Statistical_Learning_Theory\|Statistical Learning Theory]] |
| How does an LSTM remember? | [[Papers/Hochreiter1997\|Hochreiter 1997]] → [[Papers/Gers2000\|Gers 2000]] |
| How does attention work? | [[Concepts/Attention_Transformer\|Attention & Transformers]] → [[Papers/Vaswani2017_Transformer\|Vaswani 2017]] |
| Why do residual connections help? | [[Papers/He2016_ResNet\|He 2016]] → [[Concepts/Backpropagation\|Backpropagation]] (vanishing gradients) |
| How does a VAE learn a latent space? | [[Papers/Kingma2013_VAE\|Kingma 2013]] → [[Foundations/Bayesian_Inference\|Bayesian Inference]] |
| How does Adam differ from SGD? | [[Papers/Kingma2014_Adam\|Kingma 2014]] → [[Foundations/Optimisation_Theory\|Optimisation Theory]] |
| How to compare two classifiers statistically? | [[Workflows/Model_Evaluation\|Model Evaluation]] (McNemar's test) |
| How to tune the learning rate? | [[Workflows/Hyperparameter_Tuning\|Hyperparameter Tuning]] |
| Which cross-validation protocol for gestures? | [[Concepts/Cross_Validation\|Cross-Validation]] |
| What is the supervised learning framework? | [[Concepts/Supervised_Learning_Framework\|Supervised Learning Framework]] |
| What's the difference between OLS, ridge, lasso? | [[Concepts/Linear_Regression\|Linear Regression]] → [[Papers/Tibshirani1996_Lasso\|Tibshirani 1996]] |
| When to use Naive Bayes vs. logistic regression? | [[Concepts/Probabilistic_Classifiers\|Probabilistic Classifiers]] |
| How does contrastive/self-supervised learning work? | [[Concepts/Semi_Supervised_Self_Supervised\|Semi/Self-Supervised]] → [[Papers/Chen2020_SimCLR\|SimCLR]] |
| What textbook covers everything probabilistically? | [[Papers/Bishop2006_PRML\|PRML]], [[Papers/Murphy2012_MLaPP\|Murphy]], [[Papers/Hastie2009_ESL\|ESL]] |
| How does DBSCAN differ from k-means? | [[Concepts/Unsupervised_Learning\|Unsupervised Learning]] |

---

## 📐 Key Mathematical Results (Quick Reference)

| Result | Formula | Where |
|---|---|---|
| Bayes' theorem | $P(H\|E) = P(E\|H)P(H)/P(E)$ | [[Foundations/Bayesian_Inference\|Bayesian Inference]] |
| ELBO | $\mathcal{L} = \mathbb{E}[\log p_\theta(x\|z)] - \text{KL}(q_\phi \| p)$ | [[Papers/Kingma2013_VAE\|VAE]] |
| Attention | $\text{Attention}(Q,K,V) = \text{softmax}(QK^\top/\sqrt{d_k})V$ | [[Concepts/Attention_Transformer\|Transformers]] |
| Residual block | $y = F(x,W) + x$ | [[Papers/He2016_ResNet\|ResNet]] |
| Adam update | $\theta_t = \theta_{t-1} - \alpha\hat{m}_t/(\sqrt{\hat{v}_t}+\varepsilon)$ | [[Papers/Kingma2014_Adam\|Adam]] |
| VC generalisation bound | $R \leq R_{emp} + \Phi(n,h,\delta)$ | [[Foundations/Statistical_Learning_Theory\|SLT]] |
| Backpropagation delta | $\delta^{(l)} = (W^{(l+1)})^\top\delta^{(l+1)} \odot \sigma'(z^{(l)})$ | [[Concepts/Backpropagation\|Backprop]] |
| GAN minimax | $\min_G\max_D\;\mathbb{E}[\log D(x)] + \mathbb{E}[\log(1-D(G(z)))]$ | [[Papers/Goodfellow2014_GAN\|GAN]] |
