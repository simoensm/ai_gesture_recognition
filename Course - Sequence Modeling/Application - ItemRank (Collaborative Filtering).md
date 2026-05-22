# Application - ItemRank (Collaborative Filtering)

Tags: #markov-chain #recommendation #PageRank #random-walk

---

## Context

**ItemRank** is a collaborative filtering recommendation model based on **random walks with restart** on a bipartite user-item graph.

Developed by Pucci, Gori, and Faloutsos et al.

---

## Graph Structure

- **Nodes**: users + items (bipartite graph)
- **Edges**: a link exists between item $a$ and user $i$ if user $i$ bought item $a$
- **Edge weights**: e.g. purchase frequency

The [[Transition Probability Matrix]] $\mathbf{P}$ is derived from this weighted graph.

---

## Random Walker Model

A random walker on the graph at time step $k$:

$$\mathbf{x}_i(k+1) = \alpha \mathbf{P}^T \mathbf{x}_i(k) + (1-\alpha)\mathbf{v}_i$$

where:
- $\alpha$ = probability of **continuing** the walk
- $(1-\alpha)$ = probability of **restarting** the walk on a previously purchased item
- $\mathbf{v}_i$ = restart distribution: $1/n_i$ for each of the $n_i$ items bought by user $i$, else 0

---

## Steady-State Solution (Rating Scores)

As $t \to \infty$, the steady state for user $i$ is:

$$\mathbf{x}_i = (1-\alpha)(\mathbf{I} - \alpha \mathbf{P}^T)^{-1} \mathbf{v}_i$$

Items with highest $[\mathbf{x}_i]_j$ → recommended to user $i$.

---

## Connection to [[Stationary Distribution]]

The steady-state vector $\mathbf{x}_i$ is a **personalized stationary distribution** (personalized PageRank) — biased towards items already purchased by user $i$.

The global (non-personalized) case with uniform restart = standard [[Stationary Distribution]].

---

*Part of → [[Markov Chain]] | [[Stationary Distribution]] | [[MOC - Modelling Sequential Data]]*
