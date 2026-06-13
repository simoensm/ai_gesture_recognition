---
tags:
  - gesture-recognition
  - results
  - accuracy
  - comparison
  - MLSMM2154
---

# Résultats et Conclusions Clés

> **Code source des résultats :** `results/` (fichiers `.txt` et `_raw.csv`)  
> **Évaluation statistique :** [[08_Statistical_Assessment]]  
> **Protocole :** [[03_Cross_Validation_Strategy]] (Phase 2, meilleure configuration HP)

---

## Tableau Principal des Résultats

Accuracy moyenne ± écart-type sur 10 folds, sous la meilleure configuration d'hyperparamètres par méthode :

| Méthode | Domaine 1 — UI | Domaine 1 — UD | Domaine 4 — UI | Domaine 4 — UD |
|---|---|---|---|---|
| DTW | 82.7 ± 14.6 % | 99.5 ± 0.5 % | 72.6 ± 13.4 % | **99.1 ± 1.3 %** |
| Edit-Distance | 74.6 ± 20.9 % | 98.6 ± 1.0 % | 66.2 ± 13.5 % | 98.3 ± 1.5 % |
| **Three-Cents** | **96.3 ± 4.6 %** | **99.8 ± 0.4 %** | **95.1 ± 5.3 %** | 98.6 ± 1.3 % |
| BiLSTM | 85.7 ± 11.7 % | 96.0 ± 3.4 % | 73.7 ± 9.8 % | 87.9 ± 6.1 % |

**UI** = User-Independent (Leave-One-User-Out) · **UD** = User-Dependent (Leave-One-Rep-Out)

---

## Meilleures Configurations par Méthode

| Méthode | Meilleure config (Domaine 1, UI) |
|---|---|
| DTW | `k=3, w=10, no_pca` |
| Edit-Distance | `n_clusters=35, k=1, compression=True, no_pca` |
| Three-Cents | `n_points=64, no_pca` |
| BiLSTM | `target_length=32, n_units=64, dropout=0.2` |

---

## 5 Observations Clés

### 1. Three-Cents domine en User-Independent (les deux domaines)

3-Cent surpasse toutes les autres méthodes en mode UI par une large marge :
- ~**+10%** sur DTW (96.3% vs 82.7% sur Domaine 1)
- ~**+20%** sur Edit-Distance
- ~**+10%** sur BiLSTM (malgré l'avantage théorique du deep learning)

**Pourquoi 3-Cent est si fort en UI ?**
- Pas de rotation → l'orientation est discriminative (swipe gauche ≠ swipe droit)
- Scaling uniforme par longueur d'arc → préserve les proportions
- 900 templates (1-NN dans un espace normalisé) → couverture dense de l'espace des gestes
- Invariant au style individuel (normalisation géométrique forte)

### 2. Gap User-Dependent vs User-Independent

Toutes les méthodes atteignent ~99%+ en UD mais montrent beaucoup plus de variance en UI. Cela confirme que le vrai défi est la **généralisation à de nouveaux utilisateurs** — les styles de geste varient énormément d'une personne à l'autre.

```
3-Cent : 96.3% (UI) vs 99.8% (UD) → gap de ~3.5%
DTW    : 82.7% (UI) vs 99.5% (UD) → gap de ~17%
BiLSTM : 85.7% (UI) vs 96.0% (UD) → gap de ~10%
```

Plus le gap est grand, moins la méthode généralise à de nouveaux utilisateurs.

### 3. BiLSTM sous-performe par rapport aux attentes

Le deep learning n'est pas le meilleur avec seulement 1 000 séquences d'entraînement :
- 85.7% UI vs 96.3% pour 3-Cent sur Domaine 1
- Le réseau ne peut pas **tirer pleinement parti de sa capacité représentationnelle** avec si peu de données
- La généralisation à de nouveaux utilisateurs est difficile → sur-adaptation aux styles des 9 utilisateurs d'entraînement

> **Leçon :** Le deep learning nécessite des données massives pour battre des méthodes géométriques bien conçues. Avec 1 000 séquences, 3-Cent est imbattable.

### 4. L'Edit-Distance s'est améliorée significativement avec la normalisation

Avant normalisation de la distance de Levenshtein par la longueur maximale des séquences, les résultats étaient bien inférieurs. Les séquences symboliques plus longues apparaissaient disproportionnellement éloignées de leurs plus proches voisins.

**Impact de la compression** (`run-length encoding`) : amélioration significative dans certaines configurations. La compression supprime l'information de vitesse/durée (combien de temps on reste dans une région) pour ne conserver que la forme spatiale (la séquence de régions visitées).

### 5. Conclusion Statistique

**Test de Friedman (p < 0.001, α = 0.05) :**
- En mode **User-Independent** : 3-Cent est **statistiquement supérieur** à toutes les autres méthodes (confirmé par Nemenyi ET Wilcoxon-Holm)
- En mode **User-Dependent** : les différences entre DTW, Edit-Distance et 3-Cent sont **non significatives** (tous > 98%). Seul BiLSTM est significativement inférieur.

---

## Comparaison des Variances

L'écart-type reflète la variance inter-folds (= variance du comportement selon l'utilisateur test) :

| Méthode | Std UI Domaine 1 | Interprétation |
|---|---|---|
| Edit-Distance | ±20.9% | Très sensible à l'utilisateur de test |
| DTW | ±14.6% | Sensible |
| BiLSTM | ±11.7% | Modérément robuste |
| **Three-Cents** | **±4.6%** | **Le plus robuste** |

3-Cent n'est pas seulement le plus précis — c'est aussi le plus **consistant** d'un utilisateur à l'autre.

---

## Fichiers de Résultats

```
results/
├── domain1_dtw_independent.txt         # rapport humain
├── domain1_dtw_independent_raw.csv     # per-fold accuracies
├── domain1_edit-distance_independent.txt
├── domain1_edit-distance_independent_raw.csv
├── domain1_three-cent_independent.txt
├── domain1_three-cent_independent_raw.csv
├── domain1_bilstm_independent.txt
├── domain1_bilstm_independent_raw.csv
├── domain4_...                         # idem pour Domaine 4
├── statistical_tests.csv               # résultats Friedman/Nemenyi/Wilcoxon
└── cd_diagram_domain1_independent.png  # CD diagram (Demšar style)
```

---

## Connexions

- Protocole d'évaluation complet → [[03_Cross_Validation_Strategy]]
- Tests statistiques détaillés → [[08_Statistical_Assessment]]
- Décisions de conception qui expliquent ces résultats → [[10_Design_Decisions]]
- Méthode gagnante — 3-Cent → [[06_Three_Cent_Recognizer]]
