---
title: "Exploring Data Splitting Strategies for the Evaluation of Recommendation Models"
authors: "Zaiqiao Meng, Richard McCreadie, Craig Macdonald, Iadh Ounis"
year: 2020
journal: "RecSys '20 — 14th ACM Conference on Recommender Systems"
citekey: Meng2020_DataSplitting
tags:
  - recommender-systems
  - evaluation
  - data-splitting
  - methodology
  - offline-evaluation
  - confounding-variable
  - MLSMM2156
status: read
---

# Meng et al. (2020) — Stratégies de Découpage des Données pour l'Évaluation des RecSys

> **Référence :** Meng, Z., McCreadie, R., Macdonald, C. & Ounis, I. (2020). Exploring Data Splitting Strategies for the Evaluation of Recommendation Models. *RecSys '20*, ACM.

---

## 🎯 Question de Recherche

> **"Est-ce que la stratégie de découpage des données a un impact sur les résultats d'évaluation des systèmes de recommandation ?"**

La réponse des auteurs est **oui, massivement** — et c'est un problème sérieux pour la recherche actuelle.

---

## 🚨 Le Problème — Pas de Standard dans la Littérature

Quand deux chercheurs publient un article sur un nouveau modèle RecSys, ils utilisent souvent **les mêmes datasets** (ex. : MovieLens-1M) et **les mêmes métriques** (ex. : NDCG@10). On pourrait croire que leurs résultats sont comparables.

**Mais ce n'est pas le cas**, parce qu'ils n'utilisent pas forcément la même stratégie pour diviser les données en train/validation/test.

Les auteurs analysent 17 papiers récents de modèles RecSys et montrent qu'il n'existe **aucun consensus** : chaque papier utilise une stratégie différente.

> 💡 **Analogie :** C'est comme comparer deux coureurs sur un marathon, mais l'un a couru 42 km et l'autre seulement 38 km. Les temps ne sont pas comparables même si les conditions météo et le terrain étaient les mêmes.

---

## 📂 Les Quatre Stratégies de Découpage

### 1. Leave One Last (Laisser le Dernier)

On prend pour chaque utilisateur **sa dernière interaction** et on la met dans le test set. L'avant-dernière interaction va dans le validation set. Tout le reste = train.

Il y a deux variantes :

#### Leave One Last Item
- La dernière interaction = un item unique $\langle\text{user}, \text{item}\rangle$
- **Modèles qui l'utilisent :** NeuMF, SASRec, BERT4Rec, JSR, TiSASRec
- C'est la stratégie **la plus populaire** (8 papiers sur 17 dans l'analyse)

#### Leave One Last Basket/Session
- La dernière interaction = un panier complet $\langle\text{user}, [\text{item}_1, ..., \text{item}_k]\rangle$
- Utilisé pour la recommandation de courses alimentaires (grocery recommendation)
- **Modèles :** FPMC, Triple2vec, CTRec

**Avantages :**
- Maximise la taille du train set (une seule interaction par utilisateur est mise de côté)
- Simple à implémenter

**Problème majeur — le "leaking" (fuite de données) :**

Le train set contient toutes les interactions historiques, y compris des interactions qui sont postérieures dans le temps à d'autres interactions **d'autres utilisateurs** qui se retrouvent dans le test set.

En clair : **l'information sur le futur peut "fuiter" dans le modèle pendant l'entraînement**.

Exemple : L'utilisateur A a vu le film X en décembre. L'utilisateur B a vu le film X en janvier. Si on utilise Leave One Last, l'interaction de A (décembre) est dans le train, mais celle de B en janvier est dans le test. Le modèle apprend donc que X est populaire (via A) avant même que B ne l'ait vu, ce qui peut fausser les recommandations à B.

> 💡 **Analogie :** C'est comme permettre à un étudiant de voir les corrections des autres avant de passer son examen. Les résultats semblent bons, mais la méthode est biaisée.

---

### 2. Temporal Split (Découpage Temporel)

On divise les données **en fonction du temps réel**, pas en fonction des utilisateurs individuels.

#### Temporal User Split (Découpage Temporel par Utilisateur)
- Pour chaque utilisateur, on prend les **derniers X% de ses interactions** pour le test
- Exemple : les 20% dernières interactions de chaque utilisateur → test
- Similaire à Leave One Last mais plus robuste (plusieurs interactions en test, pas qu'une seule)
- **Problème :** La frontière train/test varie selon les utilisateurs → du leaking reste possible

#### Temporal Global Split (Découpage Temporel Global) ⭐ *Recommandé*
- On définit **un seul point temporel global** qui sépare train et test pour tous les utilisateurs
- Exemple : toutes les interactions avant le 1er septembre → train ; toutes après → test
- **C'est le seul découpage qui empêche vraiment le leaking**
- Les auteurs et les travaux antérieurs [Campos 2011] le considèrent comme **le plus réaliste**

**Inconvénient :** Après avoir calculé l'intersection train/test (uniquement les utilisateurs/items présents dans les deux), on perd beaucoup de données. Sur Tafeng : de 9 238 utilisateurs on passe à 1 997 seulement.

---

### 3. Random Split (Découpage Aléatoire)

On sélectionne **aléatoirement** la frontière train/test pour chaque utilisateur.

- Utilisé dans les premiers systèmes RecSys (ex. : BPR 2009)
- Progressivement abandonné au profit de Leave One Last
- **Problème principal :** Non reproductible sans partager les seeds ou les splits exacts

---

### 4. User Split (Découpage par Utilisateur)

On divise **les utilisateurs eux-mêmes** (pas leurs interactions) : certains utilisateurs → train, d'autres → test.

- Le modèle doit générer des recommandations pour des utilisateurs **jamais vus** (cold start)
- Peu utilisé car la plupart des modèles ne supportent pas le cold start
- Exemples : VAECF, SVAE

---

### Tableau Récapitulatif

| Stratégie | Réalisme | Leaking possible | Données perdues | Popularité |
|---|---|---|---|---|
| Leave One Last Item | Faible | ✅ Oui | Minimal | ⭐⭐⭐ (très courant) |
| Leave One Last Basket | Faible | ✅ Oui | Minimal | ⭐⭐ |
| Temporal User Split | Moyen | ✅ Oui | Faible | ⭐⭐ |
| **Temporal Global Split** | **Élevé** | **❌ Non** | Élevé | ⭐ (rare) |
| Random Split | Très faible | ✅ Oui | Minimal | ⭐ (ancien) |
| User Split | Spécifique (cold start) | ✅ Oui | Variable | ⭐ (rare) |

---

## 🔍 Illustration : Différence entre Temporal Global et Leave One Last

```
Leave One Last / Temporal User :
─────────────────────────────────────────►  Temps
Utilisateur 1 : [train][train][train][train] | [valid] | [test]
Utilisateur 2 : [train][train] | [valid] | [test]
                                               ↑ La frontière varie par utilisateur
                                               → Du futur peut "fuiter"

Temporal Global :
─────────────────────────────────────────►  Temps
                         ┊
Utilisateur 1 : [train]  ┊  [test]
Utilisateur 2 : [train]  ┊  [test]
                         ┊
                Frontière globale unique → Pas de leaking
```

---

## 📊 Résultats Expérimentaux

### Expérience 1 : 7 modèles sur 3 stratégies

Les auteurs testent 7 modèles (NMF, BPR, NeuMF, VAECF, NGCF, Triple2vec, VBCAR) sur deux datasets (Tafeng et Dunnhumby) avec NDCG@10 et Recall@10.

**Résultat clé :** Le classement des modèles **change significativement** selon la stratégie.

Exemple sur Dunnhumby, Recall@10 :
- Avec Leave One Last Item : NMF est le **pire** modèle (0.2498)
- Avec Leave One Last Basket : NMF monte de **3 rangs** et dépasse BPR, VAECF et NGCF

> **Conséquence :** Si un papier A teste son modèle avec Leave One Last Item et un papier B utilise Temporal Global, leur comparaison n'a **aucune validité**.

---

### Expérience 2 : 230 modèles — Corrélation de Kendall τ

Pour confirmer avec un plus grand échantillon, les auteurs génèrent 230 variantes de 3 modèles (NeuMF, VBCAR, Triple2vec) en faisant varier les hyperparamètres.

Ils calculent la **corrélation de Kendall τ** entre les classements des modèles selon les paires de stratégies :

$$\tau = \frac{(\text{paires concordantes}) - (\text{paires discordantes})}{\binom{n}{2}}$$

τ ∈ [-1, 1] : τ = 1 → classements identiques ; τ = 0 → aléatoire ; τ = -1 → ordre inverse.

**Résultats :**
- Leave One Last Item vs Leave One Last Basket (Tafeng) : τ = **0.5925**
- Leave One Last Item vs Temporal Global (Tafeng) : τ = **0.5284**
- Leave One Last Item vs Leave One Last Basket (Dunnhumby) : τ = **0.7630**
- Leave One Last Item vs Temporal Global (Dunnhumby) : τ = **0.5902**

**Interprétation :** Des valeurs entre 0.5 et 0.8 indiquent une **corrélation modérée** — loin de 1.0. Il y a beaucoup de swaps de rang, confirmant que la stratégie est une variable confondante importante.

---

### Observation : Certains Modèles Favorisés par Certaines Stratégies

- **Triple2vec** est particulièrement avantagé par Leave One Last Item
- **VBCAR** performe beaucoup mieux sous évaluation temporelle

Cela s'explique par :
1. La **distribution différente** des données train/validation/test selon la stratégie
2. L'exploitation différente du **signal temporel** (VBCAR est conçu pour capturer des patterns temporels → il est naturellement avantagé quand le test set respecte l'ordre temporel)

---

## ✅ Recommandations des Auteurs (Best Practices)

> **"Voilà ce que vous devriez faire dans vos travaux futurs :"**

**1. Rapporter explicitement la stratégie de découpage utilisée**

Inclure dans le papier : les statistiques du train/validation/test (taille, nombre d'utilisateurs/items), tout filtrage d'items/utilisateurs effectué.

**2. Rapporter les performances sous Temporal Global Splitting**

C'est le paramètre le plus réaliste. Même si d'autres stratégies sont aussi reportées pour comparaison, le Temporal Global devrait être la **référence**.

**3. Publier ses data splits**

Pour permettre la reproduction exacte des résultats par d'autres chercheurs. Sans ça, même en utilisant les mêmes datasets et métriques, les résultats ne sont pas comparables.

---

## 🔗 Connexions dans le vault

- Vue d'ensemble des méthodes d'évaluation → [[Chen2017_PerformanceEvaluation]]
- Notre protocole de découpage (K-Fold CV, LOO) → [[Workflow/03_Split_and_Evaluation]]
- Pourquoi LOO est coûteux dans notre projet → [[Workflow/15_Evaluator_vs_Ablation]]
- Novelty & Diversity → [[Castells2015_NoveltyDiversity]]
- Résultats de nos modèles → [[Workflow/08_Results_and_Decision_Making]]
