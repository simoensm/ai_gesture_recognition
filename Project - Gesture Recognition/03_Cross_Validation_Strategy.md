---
tags:
  - gesture-recognition
  - cross-validation
  - evaluation
  - user-independent
  - user-dependent
  - MLSMM2154
---

# Stratégie de Validation Croisée

> **Code :** `data/data_splitting.py`  
> **Protocole HP :** `main.py` (Phase 1 & Phase 2)  
> **Lié à :** [[02_Preprocessing]] · [[09_Results_and_Key_Findings]] · [[00_Gesture_Recognition_MOC]]

---

## Vue d'Ensemble

Deux modes de CV sont implémentés, correspondant à deux scénarios d'évaluation distincts :

| Mode | Principe | Difficulté | Scénario réel |
|---|---|---|---|
| **User-Independent (UI)** | Leave-One-User-Out | ⭐⭐⭐ Difficile | Nouvel utilisateur inconnu |
| **User-Dependent (UD)** | Leave-One-Rep-Out | ⭐ Facile | Utilisateur déjà enregistré |

Les deux sont implémentées comme des **générateurs Python** (`yield`) — ils ne matérialisent pas tous les folds simultanément en mémoire.

---

## 1. User-Independent CV (`user_independent_cv`)

### Principe : Leave-One-User-Out

Chaque fold retire **un utilisateur complet** du train.

- **10 folds**, un par sujet
- **Train :** 900 séquences (9 utilisateurs × 10 classes × 10 reps)
- **Test :** 100 séquences (1 utilisateur × 10 classes × 10 reps)
- Le sujet de test est **entièrement inconnu** du modèle — aucune fuite de son style, taille de main, ou caractéristiques motrices

### Implémentation

```python
def user_independent_cv(gestures):
    subjects = sorted(set(g["subject"] for g in gestures))
    for subject in subjects:
        train = [g for g in gestures if g["subject"] != subject]
        test  = [g for g in gestures if g["subject"] == subject]
        yield train, test, subject
```

### Pourquoi c'est la vraie évaluation ?

Un système réel doit généraliser à de **nouveaux utilisateurs** qu'il n'a jamais vus. Les styles de geste varient énormément d'une personne à l'autre (vitesse, amplitude, angle, tremblements). Le mode UI mesure exactement cette capacité de généralisation.

C'est le mode **requis par les directives du projet** pour la comparaison statistique entre méthodes.

---

## 2. User-Dependent CV (`user_dependent_cv`)

### Principe : Leave-One-Repetition-Out

Chaque fold retire **une répétition** pour chaque (sujet, type de geste).

- **10 folds**, un par index de répétition (0 à 9)
- **Train :** 900 séquences (10 utilisateurs × 10 classes × 9 reps)
- **Test :** 100 séquences (10 utilisateurs × 10 classes × 1 rep)

Le modèle a vu **toutes les répétitions précédentes** de chaque utilisateur → il peut mémoriser les styles individuels.

### Implémentation

Regroupe les gestes par `(subject, gesture_type)` dans un `defaultdict(list)`. Pour le fold `f`, le f-ième élément de chaque groupe → test, le reste → train.

### Pourquoi les résultats sont bien meilleurs en UD ?

Le modèle peut mémoriser les caractéristiques propres à chaque utilisateur (style, vitesse, amplitude). Ce n'est pas une vraie mesure de généralisation, mais utile comme **borne supérieure de performance**.

> **Exemple de gap :** 3-Cent atteint 96.3% en UI vs 99.8% en UD sur le Domaine 1. La différence de ~3.5% représente le coût de la généralisation à un nouvel utilisateur.

---

## 3. Inner Validation Split (`inner_val_split`)

### Rôle

Sélectionner les hyperparamètres **sans toucher au fold de test extérieur**.

### Protocole

- 20% du training set → validation interne
- 80% → training interne
- Seed fixe `seed=42` → reproductibilité garantie
- Utilisé **identiquement** par les quatre pipelines → comparaison équitable

```python
def inner_val_split(train_gestures, val_ratio=0.20, seed=42):
    rng     = np.random.default_rng(seed)
    indices = rng.permutation(len(train_gestures))
    n_val   = max(1, int(len(train_gestures) * val_ratio))
    inner_train = [train_gestures[i] for i in indices[n_val:]]
    inner_val   = [train_gestures[i] for i in indices[:n_val]]
    return inner_train, inner_val
```

---

## 4. Protocole de Sélection des Hyperparamètres (Deux Phases)

### Phase 1 — Recherche Globale sur Tous les Folds

Pour chaque fold extérieur :
1. Normaliser les données d'entraînement
2. Couper 20% en validation interne
3. Pour chaque combinaison HP : entraîner sur inner_train, scorer sur inner_val
4. Collecter les scores par combinaison HP

**Après tous les folds :** le meilleur HP = celui avec la **plus haute accuracy interne moyenne sur tous les folds**.

Cette sélection globale évite de choisir des HP différents pour différents folds (ce qui gonflerait artificiellement les résultats finaux).

```
Fold 1 : HP_A=0.81, HP_B=0.79, HP_C=0.83  → HP_C meilleur sur ce fold
Fold 2 : HP_A=0.85, HP_B=0.84, HP_C=0.82  → HP_A meilleur sur ce fold
...
Moyenne : HP_A=0.83, HP_B=0.81, HP_C=0.82  → HP_A sélectionné globalement
```

### Phase 2 — Évaluation Finale avec HP Fixé

Le CV est relancé avec le meilleur HP fixé. Pour chaque fold :
1. Normaliser (fit sur train)
2. Entraîner/construire les templates sur le train complet
3. Évaluer sur le test set (jamais touché en Phase 1)

**Le fold de test extérieur est UNIQUEMENT consulté en Phase 2**, préservant l'intégrité de l'évaluation.

### Pourquoi Deux Phases ?

Si on sélectionnait le meilleur HP par fold indépendamment, l'accuracy reportée inclurait le bénéfice du tuning sur des données liées au fold de test (via le split de validation dérivé de ce fold). Les deux phases garantissent **un seul HP choisi, une seule accuracy mesurée** — plus honnête et plus strict.

---

## 5. Répartition des Données par Phase

```
Dataset complet (1 000 séquences)
│
└── Fold k (10 folds en UI)
    │
    ├── TRAIN (900 séquences)  ←─ Phase 1 & Phase 2
    │   ├── inner_train (720)  ←─ Phase 1 uniquement
    │   └── inner_val   (180)  ←─ Phase 1 uniquement (sélection HP)
    │
    └── TEST  (100 séquences)  ←─ Phase 2 UNIQUEMENT (évaluation finale)
```

---

## Connexions

- Normalisation appliquée dans chaque fold → [[02_Preprocessing]]
- Résultats obtenus avec ce protocole → [[09_Results_and_Key_Findings]]
- Tests statistiques sur les résultats per-fold → [[08_Statistical_Assessment]]
- Comparaison conceptuelle avec le projet RecSys → [[Project - Movie Recommender System/Workflow/03_Split_and_Evaluation]]
