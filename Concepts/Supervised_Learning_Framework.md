---
title: "Supervised Learning — Framework & Fundamentals"
tags: [concept, supervised-learning, erm, loss-functions, generalisation, hypothesis-class, ml-foundations]
created: 2026-05-22
---

# Supervised Learning — Framework & Fundamentals

> The dominant learning paradigm: given labelled examples $(x_i, y_i)$, find a function $f: \mathcal{X} \to \mathcal{Y}$ that generalises to unseen inputs.

---

## 1. The Formal Setup

### Training data
$$\mathcal{D} = \{(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)\}, \quad (x_i, y_i) \overset{\text{iid}}{\sim} p(x, y)$$

- $x_i \in \mathcal{X}$ — input (feature vector, image, sequence…)
- $y_i \in \mathcal{Y}$ — label/target
- **iid assumption:** all examples drawn independently from the same distribution $p$

### Task taxonomy by output space

| Task | Output $\mathcal{Y}$ | Examples |
|---|---|---|
| Binary classification | $\{0, 1\}$ or $\{-1, +1\}$ | Spam/not spam, disease/healthy |
| Multiclass classification | $\{1, \ldots, K\}$ | MNIST digits, gesture classes |
| Multi-label classification | $\{0,1\}^K$ | Document tags, image attributes |
| Regression | $\mathbb{R}$ or $\mathbb{R}^d$ | House prices, trajectory prediction |
| Structured prediction | Sequences, trees, graphs | Translation, parsing, protein folding |
| Ranking | Ordered list | Search results, recommendation |

---

## 2. Empirical Risk Minimisation (ERM)

### True Risk (Population)
$$R(f) = \mathbb{E}_{(x,y)\sim p}[\ell(y, f(x))]$$
Unknown — we cannot compute this because $p$ is unknown.

### Empirical Risk (Training)
$$\hat{R}(f) = \frac{1}{n}\sum_{i=1}^n \ell(y_i, f(x_i))$$
Observable — we minimise this as a proxy for true risk.

### ERM Principle
$$\hat{f} = \arg\min_{f \in \mathcal{H}} \hat{R}(f)$$
where $\mathcal{H}$ is a **hypothesis class** (set of functions the learner can represent).

**The generalisation gap:** $R(\hat{f}) - \hat{R}(\hat{f}) \geq 0$ — training error underestimates true error. This gap shrinks as $n \to \infty$ and grows with the complexity of $\mathcal{H}$ (VC dimension theory → [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]]).

### Regularised ERM
$$\hat{f}_\lambda = \arg\min_{f \in \mathcal{H}} \hat{R}(f) + \lambda \Omega(f)$$
The regulariser $\Omega(f)$ penalises complexity. Equivalent to MAP estimation with a prior → [[Foundations/Maximum_Likelihood_Estimation|MLE & MAP]].

---

## 3. Loss Functions

The choice of $\ell$ encodes the cost of different types of errors.

### Regression Losses

| Loss | Formula | Properties |
|---|---|---|
| **Squared error (L2)** | $(y - \hat{y})^2$ | Differentiable; penalises outliers heavily |
| **Absolute error (L1)** | $|y - \hat{y}|$ | Robust to outliers; not differentiable at 0 |
| **Huber loss** | $\frac{1}{2}z^2$ if $|z|\leq\delta$, else $\delta(|z|-\delta/2)$ | Best of both: quadratic near 0, linear for outliers |
| **Log-cosh** | $\log\cosh(y-\hat{y})$ | Smooth everywhere; similar to Huber |

MSE → MLE under Gaussian noise assumption. MAE → MLE under Laplace noise.

### Classification Losses

| Loss | Formula | Properties |
|---|---|---|
| **0-1 loss** | $\mathbb{1}[\hat{y} \neq y]$ | Ideal but non-differentiable |
| **Cross-entropy** | $-\sum_k y_k \log \hat{p}_k$ | MLE under categorical; differentiable surrogate |
| **Hinge loss** | $\max(0, 1 - y\hat{f})$ | SVM; convex, not differentiable at 1 |
| **Logistic loss** | $\log(1+e^{-y\hat{f}})$ | Smooth hinge; probabilistic interpretation |
| **Focal loss** | $-(1-\hat{p})^\gamma\log\hat{p}$ | Down-weights easy examples (class imbalance) |

---

## 4. The Bias-Variance-Noise Decomposition

For squared loss, the expected test error decomposes as:
$$\mathbb{E}[(y - \hat{f}(x))^2] = \underbrace{\text{Bias}^2[\hat{f}(x)]}_{\text{systematic error}} + \underbrace{\text{Var}[\hat{f}(x)]}_{\text{estimation noise}} + \underbrace{\sigma^2_\varepsilon}_{\text{irreducible noise}}$$

- **Bias:** Error from wrong assumptions in the hypothesis class — underfitting
- **Variance:** Sensitivity to training set — overfitting
- **Irreducible noise:** Inherent randomness in $p(y|x)$

→ See [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]] for full treatment.

---

## 5. Generalisation and the Test Set Protocol

```
All data
   │
   ├── Train set (60–80%) ─── fit model parameters
   ├── Validation set (10–20%) ─── select hyperparameters / architecture
   └── Test set (10–20%) ─── report final performance (touch ONCE)
```

**Critical rules:**
- Never use test set during development — it estimates $R(\hat{f})$, not $\hat{R}(\hat{f})$
- Never tune on the validation set then report validation performance as "test" performance
- When data is scarce, use cross-validation → [[Concepts/Cross_Validation|Cross-Validation Protocols]]

---

## 6. Major Supervised Algorithms (by type)

### Linear Models
- [[Concepts/Linear_Regression|Linear Regression]] — continuous output, least squares / MLE
- [[Concepts/Logistic_Regression|Logistic Regression]] — binary/multiclass, cross-entropy
- [[Concepts/Kernel_Methods_SVM|SVM]] — margin maximisation, kernel trick

### Probabilistic / Generative
- [[Concepts/Probabilistic_Classifiers|Naive Bayes, LDA/QDA]] — model $p(x|y)$ then use Bayes
- [[Papers/Fink2014_MarkovModels|HMMs]] — sequence labelling, $p(O|\lambda)$

### Non-parametric
- [[Concepts/kNN_Classifier|k-NN]] — no training, distance-based vote

### Tree-based
- [[Concepts/Decision_Trees_Ensembles|Decision Trees, Random Forest, Gradient Boosting]]

### Neural Networks
- [[Concepts/Neural_Networks_MLP|MLP]] → [[Concepts/Convolutional_Neural_Networks|CNN]] → [[Concepts/Attention_Transformer|Transformer]]

---

## 7. Key Supervised Learning References

- [[Papers/Bishop2006_PRML|Bishop (2006)]] — comprehensive probabilistic treatment
- [[Papers/Hastie2009_ESL|Hastie, Tibshirani & Friedman (2009)]] — rigorous statistical perspective
- [[Papers/Murphy2012_MLaPP|Murphy (2012)]] — probabilistic ML, covers all algorithms
- [[Foundations/Statistical_Learning_Theory|Statistical Learning Theory]] — generalisation bounds
- [[Foundations/Maximum_Likelihood_Estimation|MLE & MAP]] — probabilistic justification for loss functions
