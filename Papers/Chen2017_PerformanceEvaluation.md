---
title: "Performance Evaluation of Recommender Systems"
authors: "Rong Chen, Qian Liu"
year: 2017
journal: "International Journal of Performability Engineering (IJPE)"
citekey: Chen2017_PerformanceEvaluation
tags:
  - recommender-systems
  - evaluation
  - metrics
  - offline-evaluation
  - information-retrieval
  - MLSMM2156
status: read
---

# Chen & Liu (2017) — Évaluation des Systèmes de Recommandation

> **Référence :** Chen, R. & Liu, Q. (2017). Performance Evaluation of Recommender Systems. *International Journal of Performability Engineering*, 13(4).

---

## 🎯 Idée Centrale

Cet article est une **revue structurée des méthodes et métriques d'évaluation** utilisées dans les systèmes de recommandation. Il organise les approches d'évaluation selon **trois grandes méthodes** (comment on évalue ?) et **quatre perspectives de métriques** (qu'est-ce qu'on mesure ?).

C'est une boîte à outils conceptuelle : avant de lancer une expérience, il faut choisir sa méthode d'évaluation ET ses métriques. Ce papier guide ce choix.

---

## 🔬 Les Trois Méthodes d'Évaluation

### 1. Évaluation hors-ligne (Offline Analytics)

On utilise un **jeu de données historiques** (ex. : MovieLens, Netflix). Aucun utilisateur réel n'est impliqué : on prédit sur des données passées et on compare aux vraies valeurs.

**Protocole typique :**
- Diviser les données en ensemble d'entraînement / test (ex. : k-fold cross-validation)
- Entraîner le modèle sur le train
- Prédire les notes sur le test
- Comparer avec les vraies notes → calculer MAE, RMSE, etc.

**Avantages :**
- Rapide, peu coûteux, reproductible
- Permet de comparer de nombreux modèles facilement

**Inconvénients :**
- Ne reflète pas forcément l'expérience réelle de l'utilisateur
- Le dataset introduit des biais (les notes absentes ne sont pas aléatoires — *missing not at random*)
- Pas de mesure de satisfaction ou de surprise

> 💡 **Analogie :** C'est comme évaluer un prof uniquement sur des examens écrits, sans jamais observer comment il interagit avec des élèves en vrai.

---

### 2. Étude utilisateur (User Study)

On recrute des **testeurs humains** qui interagissent réellement avec le système. On collecte des données qualitatives (questionnaires, interviews) et quantitatives (clics, achats).

**Exemples de protocoles :**
- A/B test en laboratoire (ex. : 20 utilisateurs testent deux interfaces)
- Questionnaire post-interaction (ex. : "Avez-vous trouvé ces recommandations utiles ?")

**Avantages :**
- Permet de mesurer la satisfaction, la confiance, la nouveauté perçue
- Plus proche de la réalité qu'une évaluation offline

**Inconvénients :**
- Coûteux (temps, recrutement)
- Échantillon souvent petit → problèmes de généralisation
- Biais liés au contexte de laboratoire (les gens se comportent différemment quand ils savent qu'ils sont observés)

---

### 3. Expérience en ligne (Online Experiment / A/B Testing)

On déploie **deux versions du système** (A et B) en production, sur des utilisateurs réels. On mesure des indicateurs comportementaux réels : taux de clics (CTR), taux de conversion, durée de session.

**Avantages :**
- Le plus réaliste — les vrais comportements d'achat/visionnage
- Large échantillon → statistiquement robuste

**Inconvénients :**
- Risque commercial (si B est moins bon que A, on perd des revenus)
- Nécessite une infrastructure de production
- Les effets peuvent prendre du temps à se manifester (ex. : recommandations qui créent de l'engagement sur le long terme)

---

### Comparatif des méthodes

| Méthode | Coût | Réalisme | Contrôle | Utilisateurs |
|---|---|---|---|---|
| Offline | Faible | Moyen | Élevé | Non |
| User study | Moyen | Moyen-élevé | Élevé | Oui (petit groupe) |
| Online (A/B) | Élevé | Très élevé | Faible | Oui (masse) |

---

## 📏 Les Quatre Perspectives de Métriques

Chen & Liu organisent les métriques selon **quatre points de vue différents**, correspondant à différentes disciplines qui s'intéressent aux RecSys.

---

### Perspective 1 — Machine Learning (Précision de Prédiction)

Ces métriques mesurent à quel point les **notes prédites** correspondent aux **notes réelles**.

Soit $Q$ l'ensemble des paires (utilisateur, item) dans l'ensemble de test, $r_{ui}$ la vraie note et $\hat{r}_{ui}$ la note prédite.

#### MAE — Mean Absolute Error (Erreur Absolue Moyenne)

$$\text{MAE} = \frac{1}{|Q|} \sum_{(u,i) \in Q} |r_{ui} - \hat{r}_{ui}|$$

- Mesure l'erreur moyenne en valeur absolue
- **Interprétation :** Si MAE = 0.8 sur une échelle 1-5, en moyenne on se trompe de 0.8 étoile
- **Simple et interprétable**, mais traite toutes les erreurs de façon égale

#### MSE — Mean Squared Error (Erreur Quadratique Moyenne)

$$\text{MSE} = \frac{1}{|Q|} \sum_{(u,i) \in Q} (r_{ui} - \hat{r}_{ui})^2$$

- Les grandes erreurs sont **pénalisées de façon disproportionnée** (effet carré)
- Utile quand les grandes erreurs sont particulièrement indésirables

#### RMSE — Root Mean Squared Error

$$\text{RMSE} = \sqrt{\frac{1}{|Q|} \sum_{(u,i) \in Q} (r_{ui} - \hat{r}_{ui})^2}$$

- Ramène le MSE dans la **même unité que les notes** (contrairement au MSE)
- C'est la métrique utilisée dans le **Netflix Prize** (2006-2009)
- Sensible aux valeurs aberrantes → si un modèle fait quelques très grosses erreurs, le RMSE le pénalise fort

> 💡 **Comparaison MAE vs RMSE :** RMSE > MAE toujours. Plus la différence est grande, plus le modèle fait des erreurs extrêmes. Si RMSE ≈ MAE, les erreurs sont uniformes.

---

### Perspective 2 — Information Retrieval (Qualité du Classement)

Ces métriques évaluent la **qualité d'une liste ordonnée** de recommandations. On ne prédit plus une note, on génère un **Top-N** d'items.

Notations :
- $N_s$ = nombre total d'items recommandés (taille de la liste)
- $N_r$ = nombre total d'items pertinents pour l'utilisateur
- $N_{rs}$ = nombre d'items pertinents parmi ceux recommandés
- $N_{rs}@n$ = nombre d'items pertinents dans les n premiers recommandés

#### Précision (Precision)

$$\text{Precision} = \frac{N_{rs}}{N_s}, \quad \text{Precision}@n = \frac{N_{rs}@n}{n}$$

- **"Parmi ce que j'ai recommandé, quelle fraction est pertinente ?"**
- Precision@10 : sur les 10 premières recommandations, combien sont bonnes ?

#### Rappel (Recall)

$$\text{Recall} = \frac{N_{rs}}{N_r}$$

- **"Parmi tous les bons items, quelle fraction ai-je réussi à recommander ?"**
- Un Recall élevé = on "ratisse large" et on rate peu de bons items

#### F-Mesure (F1-score)

$$F = \frac{2 \times \text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$

- Moyenne harmonique de Precision et Recall
- Utile quand on veut un équilibre entre les deux

> 💡 **Tension Precision/Recall :** Plus on recommande d'items, plus le Recall augmente (on couvre plus de bons items) mais la Precision baisse (on dilue avec des items moins bons). Il faut trouver le bon compromis.

#### MAP — Mean Average Precision

Pour un utilisateur $u$, l'Average Precision tient compte de **l'ordre** des items pertinents dans la liste :

$$\text{AP}(u) = \frac{1}{N_r(u)} \sum_{k=1}^{N_s} \text{Precision}@k \cdot \mathbb{1}[\text{item}_k \text{ pertinent}]$$

$$\text{MAP} = \frac{1}{|U|} \sum_{u \in U} \text{AP}(u)$$

- Un item pertinent placé en position 1 vaut plus qu'un item pertinent en position 10
- Capture la qualité de l'**ordonnancement complet**

#### ROC / AUC

- Courbe ROC : Taux de Vrais Positifs (TPR) en fonction du Taux de Faux Positifs (FPR)
- AUC = aire sous la courbe ROC ∈ [0, 1]
- AUC = 0.5 → classification aléatoire ; AUC = 1.0 → parfait
- Utile pour évaluer la **discrimination** du modèle

#### MRR — Mean Reciprocal Rank

$$\text{MRR} = \frac{1}{|Q|} \sum_{i=1}^{|Q|} \frac{1}{\text{rank}_i}$$

- $\text{rank}_i$ = position du premier item pertinent pour la requête $i$
- **"À quelle position apparaît le premier bon résultat en moyenne ?"**
- Si le premier bon item est en position 1 : contribue 1/1 = 1
- Si en position 3 : contribue 1/3 ≈ 0.33

#### SRCC — Spearman Rank Correlation Coefficient

Mesure la **corrélation entre le classement prédit et le classement réel** :

$$\rho = 1 - \frac{6 \sum d_i^2}{n(n^2-1)}$$

où $d_i$ = différence de rang pour l'item $i$.

#### nDCG — Normalized Discounted Cumulative Gain

La métrique la plus riche pour les classements :

**DCG (Discounted Cumulative Gain) :**

$$\text{DCG} = \sum_{i=1}^{N_s} \frac{\text{gain}(r_i)}{\log_2(i+1)}$$

où $\text{gain}(r_i)$ est la pertinence de l'item en position $i$ (ex. : la note réelle).

Le terme $\log_2(i+1)$ est le **facteur de dépréciation** : les items en bas de liste contribuent moins.

**nDCG :**

$$\text{nDCG} = \frac{\text{DCG}}{\text{DCG}_{\text{idéal}}}$$

- $\text{DCG}_{\text{idéal}}$ = DCG du classement parfait (items ordonnés par pertinence décroissante)
- nDCG ∈ [0, 1], normalisé pour être comparable entre utilisateurs

#### Coverage (Couverture)

$$\text{Coverage} = \frac{|\bigcup_{u \in U} I(u)|}{|I|}$$

- $I(u)$ = ensemble d'items recommandés à l'utilisateur $u$
- $|I|$ = taille totale du catalogue
- **"Quelle fraction du catalogue est jamais recommandée à quelqu'un ?"**
- Une faible coverage = le système recommande toujours les mêmes items populaires (problème de bulle de filtre)

---

### Perspective 3 — HCI / Expérience Utilisateur

Ces métriques capturent des dimensions **au-delà de la précision** qui affectent la satisfaction réelle de l'utilisateur. Elles sont moins formalisées mathématiquement mais critiques pour la pratique.

#### Diversité (Diversity)

- Mesure à quel point les items recommandés **sont différents les uns des autres**
- Formellement : opposé de la similarité intra-liste
- Si on recommande 10 films d'action similaires, la précision peut être bonne mais la diversité est faible
- Voir → [[Castells2015_NoveltyDiversity]] pour les métriques détaillées (ILD, Aggregate Diversity)

#### Confiance (Trust)

- Niveau de **confiance qu'a l'utilisateur dans les recommandations**
- Non capturé par les métriques offline
- Lié à la transparence : expliquer pourquoi un item est recommandé augmente la confiance
- Exemple : "Recommandé parce que vous avez aimé *Inception*" vs. une recommandation sans explication

#### Nouveauté (Novelty)

- Items **inconnus ou inattendus** pour l'utilisateur (qu'il n'a pas déjà vus/achetés)
- Proxy courant : **inverse popularity** = préférer les items peu populaires (l'utilisateur ne les connaît probablement pas)
- $\text{Novelty}(i) \propto -\log_2(\text{popularité}(i))$
- Un item très populaire (ex. : *Titanic*) n'est pas nouveau pour la plupart des gens

#### Sérendipité (Serendipity)

- Combinaison de **nouveauté + réaction positive surprenante**
- Un item serendipiteux = l'utilisateur ne l'aurait pas cherché mais le trouve excellent quand il le découvre
- Difficile à mesurer offline — nécessite un feedback utilisateur
- Voir → [[Castells2015_NoveltyDiversity]] pour les formules (Unexpectedness)

---

### Perspective 4 — Ingénierie Logicielle (Contraintes Système)

Ces métriques évaluent des **propriétés non-fonctionnelles** du système, indépendantes de la qualité des recommandations.

#### Temps Réel (Real-time Performance)
- Latence de la recommandation : combien de temps faut-il pour générer une liste ?
- Critique en production (ex. : Netflix doit répondre en < 200ms)
- Affecte le choix de l'algorithme : un modèle très précis mais lent peut être inutilisable

#### Robustesse (Robustness)
- Résistance aux **attaques par injection de profils** (shilling attacks)
- Un attaquant peut créer de faux utilisateurs pour faire monter un item dans les classements
- Les systèmes CF (Collaborative Filtering) sont particulièrement vulnérables
- Métrique : mesurer à quel point le classement change après l'injection de faux profils

#### Scalabilité (Scalability)
- Capacité à fonctionner avec des millions d'utilisateurs et d'items
- Certains algorithmes (ex. : User-based CF) ont une complexité $\mathcal{O}(|U|^2)$ → inutilisable à grande échelle
- Alternatives : Item-based CF, MF (Matrix Factorization), Approximate Nearest Neighbors

---

## 🗺️ Vue d'Ensemble — Architecture de l'Évaluation

```
ÉVALUATION D'UN SYSTÈME DE RECOMMANDATION
│
├── MÉTHODE D'ÉVALUATION
│   ├── Offline (dataset historique, k-fold CV)
│   ├── User Study (testeurs humains, questionnaires)
│   └── Online / A/B Testing (utilisateurs réels en production)
│
└── MÉTRIQUES
    ├── Machine Learning  → MAE, MSE, RMSE
    ├── Information Retrieval → Precision@n, Recall, F1, MAP, AUC, MRR, nDCG, Coverage
    ├── HCI / UX → Diversity, Trust, Novelty, Serendipity
    └── Software Engineering → Latency, Robustness, Scalability
```

---

## 🔗 Connexions dans le vault

- Métriques d'évaluation utilisées dans notre projet → [[Workflow/03_Split_and_Evaluation]]
- Métriques beyond-accuracy (ILD, Novelty, Coverage) → [[Workflow/09_Beyond_Accuracy]]
- Nouveauté & Diversité — article dédié → [[Castells2015_NoveltyDiversity]]
- Stratégies de splitting — impact sur les résultats → [[Meng2020_DataSplitting]]
- Évaluation CF classique (Herlocker) → [[Herlocker2004_EvaluatingCF]]
- Modèle MF (SVD, SVD++) de notre projet → [[Workflow/11_Latent_Factor_Theory]]
