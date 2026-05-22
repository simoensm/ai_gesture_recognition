Tags: #markov-chain #independence #memoryless

---

## Definition

The **Markov property** states that the probability of transitioning to the next state depends **only on the current state**, not on the sequence of states that preceded it:

$$P(s_{t+1} = j \mid s_t = i, s_{t-1}, s_{t-2}, \dots) = P(s_{t+1} = j \mid s_t = i) = p_{ij}$$

> *"The past is independent of the future given the present."*

---

## Intuition

The current state $s_t$ summarizes all information needed to predict the future. The history before $s_t$ is irrelevant.

---

## Implications

- Enables compact representation via the [[Transition Probability Matrix]] $\mathbf{P}$
- Makes computation tractable (no need to track full history)
- Basis for [[Markov Chain]] models and [[Hidden Markov Model (HMM)]]

---

## Connection to HMMs

In a [[Hidden Markov Model (HMM)]], the Markov property applies to **hidden states**:
$$P(s_t \mid s_{t-1}, s_{t-2}, \dots) = P(s_t \mid s_{t-1})$$

And observations depend only on the current hidden state:
$$P(x_t \mid s_t, s_{t-1}, \dots, x_{t-1}, \dots) = P(x_t \mid s_t)$$

---

*Part of → [[Markov Chain]] | [[MOC - Modelling Sequential Data]]*
