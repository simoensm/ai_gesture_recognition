---
title: "Optimisation Theory for ML"
tags: [foundations, optimisation, gradient-descent, sgd, adam, convexity, learning-rate]
created: 2026-05-22
---

# Optimisation Theory for ML

> [!abstract] Core role
> Training a model = minimising a loss function $\mathcal{L}(\theta)$ over parameters $\theta$. Optimisation theory explains *when* we can find the minimum and *how fast*.

**Key reference:** Ruder (2016) *An overview of gradient descent optimisation algorithms* [[Papers/Ruder2016_Optimisers|Ruder2016]]; Kingma & Ba (2014) Adam [[Papers/Kingma2014_Adam|Adam]]

---

## 1. Convexity

$f: \mathbb{R}^d \to \mathbb{R}$ is **convex** if:
$$f(\lambda x + (1-\lambda)y) \leq \lambda f(x) + (1-\lambda)f(y), \quad \forall \lambda \in [0,1]$$

**Why it matters:** For convex $f$, any local minimum = global minimum. Gradient descent is guaranteed to converge to the optimal solution.

**Strongly convex:** $f(y) \geq f(x) + \nabla f(x)^\top(y-x) + \frac{\mu}{2}\|y-x\|^2$ — unique global minimum, faster convergence.

**Neural networks are not convex** — local minima exist, but in practice loss landscapes have many equivalent minima and saddle points.

---

## 2. Gradient Descent

$$\theta_{t+1} = \theta_t - \eta \nabla_\theta \mathcal{L}(\theta_t)$$

- $\eta$: learning rate (step size)
- $\nabla_\theta \mathcal{L}$: gradient computed on the **full dataset**

**Convergence (convex, $L$-smooth):** $\mathcal{L}(\theta_t) - \mathcal{L}^* \leq \mathcal{O}(1/t)$ with $\eta \leq 1/L$

**Problem:** Full-batch gradient is expensive for large datasets.

---

## 3. Stochastic Gradient Descent (SGD)

Use a single sample (or mini-batch of size $B$) per step:
$$\theta_{t+1} = \theta_t - \eta \nabla_\theta \mathcal{L}_{i_t}(\theta_t)$$

**Mini-batch SGD** ($B$ samples): gradient estimator with noise $\propto \sigma/\sqrt{B}$.

**Advantages:**
- Much cheaper per step
- Noise can escape sharp minima → better generalisation (implicit regularisation)
- Works for streaming / online data

**Disadvantage:** Requires careful learning rate tuning.

---

## 4. Momentum

Accelerates convergence by accumulating a velocity vector:
$$v_{t+1} = \beta v_t + (1-\beta)\nabla_\theta \mathcal{L}$$
$$\theta_{t+1} = \theta_t - \eta v_{t+1}$$

Typical $\beta = 0.9$. Dampens oscillations, accelerates along consistent gradient directions.

**Nesterov Momentum:** Compute gradient at the anticipated next point → faster theoretical convergence.

---

## 5. Adaptive Learning Rates

| Algorithm | Update rule | Key property |
|---|---|---|
| **AdaGrad** (2011) | $\eta_i = \eta / \sqrt{G_{ii} + \varepsilon}$ | Per-parameter LR; good for sparse gradients |
| **RMSprop** (2012) | $\eta_i = \eta / \sqrt{v_i + \varepsilon}$, $v_i = \beta v_i + (1-\beta)g_i^2$ | Exponential decay of squared gradients |
| **Adam** (2014) | See below | Combines momentum + RMSprop + bias correction |

### Adam (Kingma & Ba, 2014)

The de facto standard optimiser for deep learning:
$$m_t = \beta_1 m_{t-1} + (1-\beta_1)g_t \quad \text{(1st moment / momentum)}$$
$$v_t = \beta_2 v_{t-1} + (1-\beta_2)g_t^2 \quad \text{(2nd moment / RMSprop)}$$
$$\hat{m}_t = m_t/(1-\beta_1^t), \quad \hat{v}_t = v_t/(1-\beta_2^t) \quad \text{(bias correction)}$$
$$\theta_{t+1} = \theta_t - \eta \hat{m}_t / (\sqrt{\hat{v}_t} + \varepsilon)$$

Defaults: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\varepsilon = 10^{-8}$, $\eta = 10^{-3}$.

**AdamW:** Decouples weight decay from the adaptive update — the correct way to apply L2 regularisation with Adam. Used in the BiLSTM implementation (→ [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]]).

---

## 6. Learning Rate Scheduling

| Schedule | Rule | When to use |
|---|---|---|
| **Step decay** | $\eta \leftarrow \eta \cdot \gamma$ every $k$ epochs | Simple, common |
| **Cosine annealing** | $\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max}-\eta_{\min})(1+\cos(\pi t/T))$ | Vision, NLP transformers |
| **Warmup** | Linear increase for first $k$ steps | Transformers (essential) |
| **ReduceLROnPlateau** | Halve LR when val loss stagnates | General purpose |

---

## 7. Gradient Clipping

$$g \leftarrow g \cdot \min\left(1, \frac{c}{\|g\|}\right)$$

Prevents exploding gradients in RNNs/LSTMs. Used with `max_norm=1.0` in the BiLSTM trainer (→ [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]]).

---

## 8. Second-Order Methods

**Newton's method:** $\theta_{t+1} = \theta_t - H^{-1}\nabla\mathcal{L}$ where $H = \nabla^2\mathcal{L}$ is the Hessian.
- Quadratic convergence near optimum
- Computing $H^{-1}$ costs $\mathcal{O}(d^3)$ — prohibitive for neural nets

**L-BFGS:** Limited-memory approximation of the Hessian inverse. Useful for small networks or convex problems.

---

## Connections

- Gradients → [[Linear_Algebra_for_ML]] (Jacobians)
- Backpropagation computes gradients → [[Concepts/Backpropagation|Backpropagation]]
- Loss functions being optimised → [[Maximum_Likelihood_Estimation]]
- Adam in BiLSTM → [[Project - Gesture Recognition/04_BiLSTM|04_BiLSTM]]
- Convergence proofs use → [[Statistical_Learning_Theory]]
