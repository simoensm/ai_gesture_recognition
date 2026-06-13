---
tags:
  - gesture-recognition
  - preprocessing
  - normalization
  - pca
  - data-leakage
  - MLSMM2154
---

# Prétraitement — Normalisation & PCA

> **Code :** `data/data_preparation.py`  
> **Lié à :** [[03_Cross_Validation_Strategy]] · [[01_Data_and_Datasets]] · [[00_Gesture_Recognition_MOC]]

---

## Vue d'Ensemble

Deux étapes de prétraitement sont appliquées, dans cet ordre :

```
Gestes bruts (x, y, z)
  → fit_normalizer(train) → apply_normalizer(train, test)   [obligatoire]
  → fit_pca_per_gesture(train) → apply_pca_per_gesture(...)  [optionnel, grille HP]
  → Gestes normalisés + projetés
```

**Règle d'or anti-leakage :** tout `fit_*` n'utilise QUE les données d'entraînement. Les statistiques apprises sont ensuite appliquées (`apply_*`) aux données de test sans aucune re-estimation.

---

## 1. Normalisation Z-Score

### Méthode

**Standardisation Z-score** appliquée par dimension spatiale (x, y, z) **indépendamment**.

$$x'_{ij} = \frac{x_{ij} - \mu_j}{\sigma_j}, \quad j \in \{x, y, z\}$$

où $\mu_j$ et $\sigma_j$ sont calculés sur **tous les points** de **toutes les trajectoires du train set** empilés en une seule matrice.

### Protocole (Deux Fonctions Séparées)

```python
# Étape 1 — sur les données d'entraînement seulement
mean, std = fit_normalizer(train_gestures)
# mean.shape = (3,), std.shape = (3,)

# Étape 2 — appliquer aux deux ensembles
train_norm = apply_normalizer(train_gestures, mean, std)
test_norm  = apply_normalizer(test_gestures, mean, std)
```

Le test set est normalisé avec les **statistiques du train**, jamais les siennes propres.

### Pourquoi Normaliser ?

Les coordonnées $(x, y, z)$ brutes dépendent du setup d'enregistrement : distance par rapport au capteur, échelle de la pièce, calibration. Sans normalisation :
- Un geste réalisé à grande échelle apparaît plus "distant" d'un template réalisé à petite échelle → fausses mauvaises classifications
- Les distances DTW et edit sont biaisées par l'échelle absolue

La normalisation supprime cette ambiguïté d'échelle tout en conservant la forme relative du geste.

### Pourquoi par Dimension Indépendamment ?

Les axes x, y, z n'ont pas nécessairement la même variance (ex. : mouvements surtout horizontaux → $\sigma_x > \sigma_z$). Normaliser par dimension rend chaque axe comparable en variance.

---

## 2. PCA par Classe (Optionnel)

### Méthode

**PCA par type de geste.** Un `sklearn.PCA` distinct est entraîné pour chaque classe de geste, sur les données d'entraînement de cette classe uniquement.

**Options dans la grille :** `no_pca`, 2 composantes, 3 composantes.

### Protocole

```python
# Étape 1 — fit sur train uniquement, une PCA par classe
pcas = fit_pca_per_gesture(train_gestures, n_components)
# pcas = {gesture_type: sklearn.PCA fitted on that class's train points}

# Étape 2 — appliquer à chaque geste selon sa classe
train_pca = apply_pca_per_gesture(train_gestures, pcas)
test_pca  = apply_pca_per_gesture(test_gestures, pcas)
```

Chaque geste dans le test set est projeté avec la PCA de **sa propre classe** (en utilisant les axes appris sur le train de cette classe).

### Pourquoi PCA par Classe plutôt que PCA Globale ?

Chaque classe de geste a une **structure géométrique caractéristique** :
- Un chiffre "1" → presque 1D (geste très linéaire)
- Un cercle → 2D (il décrit un plan)
- Une sphère → 3D (occupe les trois dimensions)

Une PCA globale calcule un sous-espace qui **moyenne** toutes les classes → elle écrase potentiellement la variance discriminante. Une PCA par classe projette chaque geste sur **son propre sous-espace caractéristique**.

### Pourquoi Proposer `no_pca` comme Option ?

Pour le BiLSTM, la PCA n'est **pas appliquée** : le réseau est censé apprendre sa propre représentation interne depuis les trajectoires normalisées brutes. La PCA interférerait avec cet apprentissage.

Pour DTW, Edit Distance et 3-Cent, `no_pca` est aussi une option valide : si la variance discriminante est répartie dans les 3 dimensions, la projection PCA pourrait en éliminer.

### Valeur Ajoutée de la PCA dans ce Contexte

La PCA peut améliorer la robustesse en :
1. **Débruitant** : les dernières composantes principales capturent souvent le bruit
2. **Réduisant la dimensionnalité** : avec 2D ou 3D, les distances DTW/edit deviennent moins coûteuses à calculer
3. **Alignant** les gestes de la même classe dans un sous-espace commun

---

## 3. Ordre d'Application dans le Pipeline Complet

```
Données brutes
     │
     ▼  [fold-level : une fois par fold de CV]
fit_normalizer(inner_train)
apply_normalizer(inner_train, inner_val, test)
     │
     ▼  [si PCA activée]
fit_pca_per_gesture(inner_train, n_components)
apply_pca_per_gesture(inner_train, inner_val, test)
     │
     ▼
Pipeline spécifique (DTW / Edit / 3-Cent / BiLSTM)
```

---

## 4. Prévention des Fuites de Données (Data Leakage)

| Risque | Comment il est évité |
|---|---|
| Normalisation avec stats du test | `fit_normalizer` est appelé uniquement sur `train_gestures` |
| PCA avec variance du test | `fit_pca_per_gesture` est appelé uniquement sur `inner_train` |
| Hyperparamètres choisis sur le test | Sélection sur `inner_val` (20% du train), jamais sur le test |
| Stats du fold de test influençant le modèle | Le fold de test n'est touché qu'en Phase 2 de l'évaluation |

> **Règle :** Tout ce qu'on "fit" ou "apprend" doit l'être sur les données d'entraînement uniquement. Le test set simule le monde réel — il est inconnu au moment du déploiement.

---

## Connexions

- Protocole de CV (Phase 1 / Phase 2) → [[03_Cross_Validation_Strategy]]
- Utilisation dans DTW → [[04_DTW_kNN]]
- Utilisation dans Edit Distance → [[05_Edit_Distance_kNN]]
- Utilisation dans 3-Cent → [[06_Three_Cent_Recognizer]]
- Utilisation dans BiLSTM → [[07_BiLSTM]]
