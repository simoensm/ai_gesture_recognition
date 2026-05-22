---
title: "Information Theory"
tags: [foundations, information-theory, entropy, kl-divergence, mutual-information, cross-entropy]
created: 2026-05-22
---

# Information Theory

> [!abstract] Why ML needs information theory
> Cross-entropy loss = NLL = information-theoretic compression cost. KL divergence = the gap between your model and reality. Mutual information = feature relevance. Shannon's 1948 framework underpins all of it.

**Key reference:** Cover & Thomas (2006) *Elements of Information Theory* — [[Papers/Cover2006_InfoTheory|Cover2006]]

---

## 1. Shannon Entropy

The **entropy** of a discrete random variable $X$ with distribution $P$:
$$H(X) = -\sum_x P(x) \log P(x) = \mathbb{E}_P[-\log P(X)]$$

- Measures **uncertainty** or **information content**
- Units: bits (log base 2) or nats (natural log)
- $H(X) \geq 0$, with equality iff $X$ is deterministic
- Maximum entropy: uniform distribution → $H_{\max} = \log |\mathcal{X}|$

**Continuous (differential entropy):**
$$h(X) = -\int p(x) \log p(x)\, dx$$

Note: differential entropy can be negative (unlike discrete entropy).

---

## 2. KL Divergence (Relative Entropy)

$$\text{KL}(P \| Q) = \sum_x P(x) \log \frac{P(x)}{Q(x)} = \mathbb{E}_P\left[\log \frac{P(X)}{Q(X)}\right]$$

**Properties:**
- $\text{KL}(P \| Q) \geq 0$ (Gibbs' inequality), = 0 iff $P = Q$ a.e.
- **Asymmetric:** $\text{KL}(P \| Q) \neq \text{KL}(Q \| P)$ in general
- Not a metric (no triangle inequality)

**ML interpretation:**  
"Extra bits needed to encode samples from $P$ using a code optimised for $Q$."

**Connection to MLE:** $\hat{\theta}_{\text{MLE}} = \arg\min_\theta \text{KL}(\hat{P}_{\text{data}} \| P_\theta)$

---

## 3. Cross-Entropy

$$H(P, Q) = -\sum_x P(x) \log Q(x) = H(P) + \text{KL}(P \| Q)$$

**The cross-entropy loss in classification:**
- $P$ = one-hot true distribution
- $Q$ = model's predicted probabilities (softmax)
- $H(P, Q) = -\log Q(y_{\text{true}})$ — the standard categorical cross-entropy loss

When $H(P)$ is fixed (true labels are given), minimising $H(P,Q)$ = minimising $\text{KL}(P\|Q)$ = MLE.

---

## 4. Mutual Information

$$I(X; Y) = \text{KL}(P(X,Y) \| P(X)P(Y)) = H(X) - H(X \mid Y) = H(Y) - H(Y \mid X)$$

- Measures how much knowing $Y$ reduces uncertainty about $X$
- $I(X;Y) = 0$ iff $X \perp Y$
- Symmetric: $I(X;Y) = I(Y;X)$

**ML applications:**
- Feature selection: select features $X_j$ that maximise $I(X_j; Y)$
- Decision trees (Information Gain = mutual information)
- Representation learning: maximise $I(\text{representation}; \text{input})$

---

## 5. Conditional Entropy & Chain Rule

$$H(X \mid Y) = -\sum_{x,y} P(x,y) \log P(x \mid y)$$

**Chain rule of entropy:**
$$H(X_1, \ldots, X_n) = \sum_{i=1}^n H(X_i \mid X_1, \ldots, X_{i-1})$$

---

## 6. Jensen-Shannon Divergence (JSD)

A symmetrised, bounded version of KL:
$$\text{JSD}(P \| Q) = \frac{1}{2}\text{KL}(P \| M) + \frac{1}{2}\text{KL}(Q \| M), \quad M = \frac{P+Q}{2}$$

- $0 \leq \text{JSD} \leq 1$ (in bits)
- Symmetric
- **GAN connection:** The original GAN loss (Goodfellow 2014) minimises the JSD between real and generated distributions — see [[Papers/Goodfellow2014_GAN|GAN paper]]

---

## 7. Summary: Key Relationships

```
H(P)          ← pure uncertainty of P
H(P, Q)       ← encoding P using Q (cross-entropy loss)
KL(P‖Q)       ← gap: H(P,Q) − H(P)  (extra cost from wrong model)
I(X;Y)        ← shared information / dependence
JSD(P‖Q)      ← symmetric distance between distributions
```

---

## Connections

- Cross-entropy = standard classification loss → [[Concepts/Neural_Networks_MLP|Neural Networks]]
- KL divergence = ELBO term → [[Papers/Kingma2013_VAE|VAE]]
- KL divergence = MLE → [[Maximum_Likelihood_Estimation]]
- Information gain = decision tree splitting criterion → [[Concepts/Decision_Trees_Ensembles|Decision Trees]]
- JSD = original GAN loss → [[Papers/Goodfellow2014_GAN|GAN]]
