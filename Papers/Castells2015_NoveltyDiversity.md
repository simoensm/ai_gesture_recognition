---
title: "Novelty and Diversity in Recommender Systems"
authors: "Pablo Castells, Neil Hurley, Shlomo Vargas"
year: 2015
journal: "Recommender Systems Handbook (Chapter 26)"
citekey: Castells2015_NoveltyDiversity
tags:
  - recommender-systems
  - novelty
  - diversity
  - serendipity
  - evaluation
  - beyond-accuracy
  - MLSMM2156
status: read
---

# Castells, Hurley & Vargas (2015) — Nouveauté et Diversité dans les RecSys

> **Référence :** Castells, P., Hurley, N. & Vargas, S. (2015). Novelty and Diversity in Recommender Systems. In *Recommender Systems Handbook* (Chapter 26). Springer.

---

## 🎯 Idée Centrale

Ce chapitre répond à une question fondamentale : **un bon système de recommandation est-il forcément celui qui prédit le mieux les notes ?**

La réponse des auteurs est **non**. Un système trop centré sur la précision tombe dans le piège de la **bulle de filtre** : il recommande toujours les mêmes items populaires que l'utilisateur connaît déjà, ce qui réduit la valeur ajoutée de la recommandation.

Le chapitre propose un **cadre conceptuel unifié** pour définir et mesurer la nouveauté, la diversité et la sérendipité, et présente des algorithmes pour les améliorer.

---

## 🧠 Concepts Fondamentaux

### La Nouveauté (Novelty)

> "La nouveauté est la **différence entre l'expérience présente et l'expérience passée** de l'utilisateur."

Formellement, la nouveauté est relative à un **contexte θ** (l'historique de l'utilisateur, ses préférences connues, sa culture générale...).

Un item $i$ est **nouveau** pour l'utilisateur $u$ si $u$ n'a pas (ou peu) été exposé à $i$ dans le contexte $\theta$.

**Subtilité :** la nouveauté n'est pas la même chose que l'impopularité. Un item très populaire peut être nouveau pour un utilisateur en particulier s'il ne l'a pas encore vu. Mais en pratique, on utilise souvent l'impopularité comme proxy.

**Proxy courant — Inverse User Frequency (IUF) :**

$$\text{Novelty}(i) = -\log_2\left(\frac{|U_i|}{|U|}\right)$$

où $|U_i|$ = nombre d'utilisateurs ayant interagi avec $i$, et $|U|$ = nombre total d'utilisateurs.

> 💡 **Analogie :** Recommander *Titanic* à quelqu'un est peu nouveau (tout le monde l'a vu). Recommander un film d'art et essai des années 70 est plus nouveau. La nouveauté récompense la "découverte".

---

### La Diversité (Diversity)

> "La diversité mesure les **différences internes** au sein d'une liste de recommandations."

Deux types de diversité sont distingués :

**Diversité intra-liste :** À quel point les items recommandés à un même utilisateur sont-ils différents les uns des autres ?

**Diversité inter-utilisateurs :** À quel point les recommandations varient-elles d'un utilisateur à l'autre ?

> 💡 **Analogie :** Un buffet diversifié offre des plats de différentes cuisines. Un buffet peu diversifié propose 10 variantes de pizza. Les deux peuvent être "bons" au sens de la précision, mais le premier offre plus de valeur.

---

### La Sérendipité (Serendipity)

> "La sérendipité = nouveauté + réaction émotionnelle positive inattendue."

Un item sérendipiteux est **à la fois nouveau ET surprenant positivement**. L'utilisateur ne l'aurait pas cherché, mais quand il le découvre, il est ravi.

**Différence avec la simple nouveauté :** Un item peut être nouveau mais peu intéressant (ex. : recommander un film de genre que l'utilisateur déteste). La sérendipité exige la pertinence en plus de la nouveauté.

---

### Le Problème de la Bulle de Filtre (Filter Bubble)

Eli Pariser (2011) a popularisé ce concept : les algorithmes de recommandation, en cherchant à maximiser la précision, finissent par enfermer l'utilisateur dans une **bulle d'informations**.

- L'utilisateur ne voit que ce qui confirme ses goûts existants
- Les items populaires ("blockbusters") sont sur-recommandés
- Les items de la **longue traîne** (items moins populaires mais potentiellement très appréciés par certains) sont ignorés
- Conséquence commerciale : les plateformes ratent des ventes potentielles de niche

**La longue traîne (Long Tail) :** Stratégie qui consiste à vendre beaucoup d'items peu populaires (plutôt que peu d'items très populaires). La diversité et la nouveauté permettent d'exploiter cette longue traîne.

---

## 📐 Métriques de Nouveauté

### MIUF — Mean Inverse User Frequency

$$\text{MIUF}(R_u) = -\frac{1}{|R_u|} \sum_{i \in R_u} \log_2\left(\frac{|U_i|}{|U|}\right)$$

- $R_u$ = liste de recommandations pour l'utilisateur $u$
- Plus $|U_i|$ est petit (item peu populaire), plus sa contribution à MIUF est grande
- Une MIUF élevée = on recommande des items peu populaires = bonne nouveauté de longue traîne
- **Limite :** ne tient pas compte de si l'utilisateur a déjà vu l'item

**Interprétation :** Un item vu par 10% des utilisateurs a $-\log_2(0.1) \approx 3.32$ bits de nouveauté. Un item vu par 50% en a $-\log_2(0.5) = 1$ bit. Les items rares sont exponentiellement plus "nouveaux".

---

### Unexpectedness (Innattendu)

$$\text{Unexpectedness}(R_u) = \frac{|R_u \setminus EX_u|}{|R_u|}$$

où $EX_u$ = ensemble des items **attendus** (que l'utilisateur s'attendrait à voir recommandé, par exemple les items très populaires ou les suites directes de films qu'il a vus).

- **Unexpectedness = fraction des recommandations qui surprennent l'utilisateur**
- Si un modèle naïf basé sur la popularité aurait recommandé les mêmes items, Unexpectedness ≈ 0

---

## 📐 Métriques de Diversité

### ILD — Intra-List Diversity (Diversité Intra-Liste)

$$\text{ILD}(R_u) = \frac{1}{|R_u|(|R_u|-1)} \sum_{i \in R_u} \sum_{j \in R_u, j \neq i} d(i, j)$$

- $d(i, j)$ = distance (dissimilarité) entre les items $i$ et $j$ (ex. : 1 - cosine similarity des features)
- Moyenne des dissimilarités pour toutes les paires d'items dans la liste
- ILD ∈ [0, 1] si $d$ est normalisée
- **ILD élevée** = items très différents les uns des autres = diversité intra-liste élevée

> 💡 **Exemple concret :** Si $R_u = \{\text{Star Wars, Interstellar, The Godfather}\}$ : les deux premiers sont de la SF, le troisième est un drame mafieux → les paires (Star Wars, Godfather) et (Interstellar, Godfather) ont une grande distance → ILD élevée.

---

### Aggregate Diversity (Diversité Agrégée)

$$\text{Aggdiv} = \left|\bigcup_{u \in U} R_u\right|$$

- Nombre total d'**items distincts** recommandés à travers tous les utilisateurs
- Si le système recommande toujours les mêmes 100 films à tout le monde alors que le catalogue en contient 10 000, Aggdiv = 100 (très faible)
- Equivalent à la **Coverage** de Chen & Liu → voir [[Chen2017_PerformanceEvaluation]]

---

### IUD — Inter-User Diversity (Diversité Inter-Utilisateurs)

$$\text{IUD} = \frac{1}{|U|(|U|-1)} \sum_{u \in U} \sum_{v \in U, v \neq u} d(R_u, R_v)$$

- Mesure à quel point les recommandations **varient d'un utilisateur à l'autre**
- $d(R_u, R_v)$ peut être la distance de Jaccard entre les deux listes : $1 - |R_u \cap R_v| / |R_u \cup R_v|$
- IUD élevée = personnalisation forte (chaque utilisateur reçoit des recommandations différentes)

---

### ISD — Inter-System Diversity

- Compare la diversité entre **deux systèmes de recommandation différents**
- Utile pour vérifier si un nouveau modèle apporte réellement une valeur différente par rapport à l'existant

---

### TD — Temporal Diversity (Diversité Temporelle)

- Mesure à quel point les recommandations **changent au fil du temps** pour un même utilisateur
- Important pour éviter qu'un utilisateur reçoive toujours les mêmes suggestions à chaque visite

---

## 🧮 Cadre Unifié — La Métrique Générale de Nouveauté

Les auteurs proposent une formulation unifiée qui subsume toutes les métriques de nouveauté/diversité :

$$m(\mathbf{R} | \theta) = \frac{1}{|R|} \sum_{i \in R} \text{nov}(i | \theta)$$

où :
- $R$ = liste de recommandations
- $\theta$ = contexte de référence (historique de l'utilisateur, état du monde, autre liste...)
- $\text{nov}(i | \theta)$ = mesure de nouveauté de l'item $i$ relativement au contexte $\theta$

**Ce cadre subsume :**
- La **nouveauté individuelle** : $\theta$ = historique de l'utilisateur
- La **diversité intra-liste** : $\theta$ = les autres items de la même liste
- La **diversité inter-utilisateurs** : $\theta$ = les listes des autres utilisateurs

---

## 🔧 Méthodes pour Améliorer la Nouveauté et la Diversité

### 1. Re-ranking avec MMR (Maximal Marginal Relevance)

L'algorithme greedy MMR construit la liste de recommandations en maximisant simultanément la pertinence et la diversité :

$$g(R, \lambda) = (1 - \lambda) \cdot \frac{1}{|R|} \sum_{i \in R} f_{\text{rel}}(i) + \lambda \cdot \text{div}(R)$$

- $f_{\text{rel}}(i)$ = score de pertinence de l'item $i$ (ex. : note prédite)
- $\text{div}(R)$ = diversité de la liste (ex. : ILD)
- $\lambda \in [0, 1]$ = paramètre de trade-off entre pertinence et diversité

**Algorithme greedy :**
1. Initialiser $R = \emptyset$
2. À chaque étape, ajouter l'item $i$ qui maximise $g(R \cup \{i\}, \lambda)$
3. Répéter jusqu'à atteindre la taille souhaitée

Quand $\lambda = 0$ → sélection purement basée sur la pertinence (classement standard).  
Quand $\lambda = 1$ → sélection purement basée sur la diversité.

> 💡 **Intuition :** À chaque étape, MMR choisit l'item qui apporte le meilleur compromis entre "être bon" et "être différent de ce qu'on a déjà sélectionné".

---

### 2. Approches basées sur le Clustering

- Regrouper les items en clusters thématiques (ex. : genres de films)
- Assurer que la liste finale contient des items de **plusieurs clusters différents**
- Plus simple que MMR mais moins finement contrôlable

---

### 3. Fusion / Hybridation

- Combiner un modèle précis (ex. : MF) avec un modèle favorisant la nouveauté (ex. : CF sur la longue traîne)
- La pondération permet de régler le curseur précision/diversité

---

### 4. Learning to Rank avec Diversité

- Intégrer directement la diversité comme **objectif d'apprentissage** dans le modèle
- Cadre de Diversified Ranking : la fonction de perte intègre à la fois la pertinence et la diversité
- Plus complexe à optimiser mais donne en théorie les meilleurs résultats

---

## 📊 Résultats Empiriques (MovieLens 1M)

Les auteurs comparent plusieurs familles de modèles sur MovieLens 1M en termes de précision (nDCG) et de diversité (Aggregate Diversity, Novelty).

| Modèle | nDCG | Aggdiv | Long-tail Novelty |
|---|---|---|---|
| **Matrix Factorization (MF)** | **0.3161** | **0.2817** | Moyen |
| Content-Based (CB) | 0.28 | 0.24 | **Élevé** |
| Popularity-Based (PopRec) | 0.30 | 0.05 | **Très faible** |
| Random | 0.05 | **Très élevé** | Élevé |

**Observations clés :**
- Le modèle basé sur la **popularité (PopRec)** a une bonne précision mais une diversité catastrophique (recommande toujours les mêmes blockbusters)
- La **recommandation aléatoire** a la plus grande diversité, mais une précision quasi nulle
- **MF** offre le meilleur compromis : précision ET diversité correctes
- Le **Content-Based** est excellent pour la nouveauté de longue traîne (il recommande des items de niche thématiquement cohérents)

> 💡 **Leçon :** Il n'existe pas de modèle "meilleur sur tous les axes". Le choix dépend des objectifs business (exploiter la longue traîne ? maximiser les clics ? fidéliser l'utilisateur ?).

---

## 🔗 Connexions dans le vault

- Métriques d'évaluation globales → [[Chen2017_PerformanceEvaluation]]
- ILD, Coverage, Hit Rate dans notre projet → [[Workflow/09_Beyond_Accuracy]]
- Modèle MF (SVD) utilisé dans notre projet → [[Workflow/11_Latent_Factor_Theory]]
- Évaluation des RecSys (Herlocker 2004) → [[Herlocker2004_EvaluatingCF]]
- Stratégies de splitting → [[Meng2020_DataSplitting]]
