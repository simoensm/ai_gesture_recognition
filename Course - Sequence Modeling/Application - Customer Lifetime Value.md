---
tags:
  - markov-chain
  - application
  - marketing
  - CLV
  - sequence-modeling
---
# Application - Customer Lifetime Value

Tags: #markov-chain #application #marketing #CLV

---

## Context

A [[Markov Chain]] model for customer behaviour:
- States = **customer segments** (e.g. RFM clusters: Recency, Frequency, Monetary value)
- The **last state** ($n$-th) = lost customers → **absorbing state**, benefit = 0
- Each month, customers move between segments

---

## Model Setup

**Transition probabilities:** estimated from observed monthly movements between clusters (empirical frequencies).

**Profit per segment:** $m_i$ (€/month) for customers in state $i$ — may be negative.

**Discounting factor:** $0 < \gamma < 1$ (accounts for inflation / time value of money).

---

## Expected Profit on Infinite Horizon

Starting from segment $i$, the discounted expected profit is:

$$\mu = \sum_{t=0}^{\infty} \gamma^t \sum_{i=1}^{n} x_i(t) \, m_i$$

In matrix form, this simplifies to:

$$\mu = \mathbf{x}^T(0) \left(\sum_{t=0}^{\infty} \gamma^t \mathbf{P}^t\right) \mathbf{m} = \mathbf{x}^T(0)(\mathbf{I} - \gamma\mathbf{P})^{-1} \mathbf{m}$$

---

## Connection to Absorbing Chain Theory

When $\gamma = 1$ (no discounting) and there is a lost-customer absorbing state, this is equivalent to computing the [[Expected Cost Until Absorption]] with profit instead of cost.

---

## Practical Use

This gives the **customer lifetime value (CLV)**: the expected total profit from a group of customers until they leave the company.

Used for:
- Targeted marketing decisions
- Prioritizing high-value customer segments
- Budget allocation for retention campaigns

---

*Part of → [[Absorbing Markov Chain]] | [[Markov Chain]] | [[MOC - Modelling Sequential Data]]*
