---
tags:
  - gesture-recognition
  - design-decisions
  - architecture
  - MLSMM2154
---

# Décisions de Conception — Justifications Techniques

> **Vue d'ensemble des résultats :** [[09_Results_and_Key_Findings]]  
> **Lié à :** [[00_Gesture_Recognition_MOC]]

Ce document catalogue les décisions majeures prises tout au long du projet et la raison précise derrière chacune.

---

## Données & Splits

| Décision | Raison |
|---|---|
| Ignorer la colonne timestamp | La tâche cible la forme spatiale, pas la vitesse. La normalisation de vitesse est mieux gérée par le rééchantillonnage. |
| CV basé sur des générateurs Python (`yield`) | Évite de matérialiser tous les 10 folds simultanément en mémoire |
| 20% de split de validation interne avec seed=42 | Sélection HP reproductible ; même protocole pour toutes les méthodes → comparaison équitable |
| Sélection HP globale sur tous les folds (Phase 1) | Évite que le tuning HP par fold gonfle artificiellement l'accuracy reportée |

---

## Prétraitement

| Décision | Raison |
|---|---|
| Normalisation Z-score | Supprime l'ambiguïté de position et d'échelle introduite par les différences de setup d'enregistrement |
| Fit du normaliseur sur le train uniquement | Prévention standard du data leakage ; les stats du test sont inconnues au déploiement |
| PCA par classe plutôt que globale | Chaque classe de geste a une structure géométrique caractéristique distincte |
| PCA appliquée par classe sur train uniquement | Empêche la variance du test d'influencer les axes de projection |
| Pas de PCA pour BiLSTM | Le réseau apprend sa propre représentation ; la PCA interférerait avec cet apprentissage |

---

## Algorithmes — DTW

| Décision | Raison |
|---|---|
| DTW normalisé par la longueur du chemin de warping | Supprime le biais vers les séquences plus courtes dans la comparaison de distance k-NN |
| Fenêtre Sakoe-Chiba comme hyperparamètre optionnel | Contraint le warping pour éviter les alignements pathologiques ; valeur optimale dépend des données |
| Cache des distances DTW par fold | Évite de recalculer les distances DTW pour différentes valeurs de k |
| Numba JIT (`@jit(nopython=True)`) | Sans Numba, la recherche de grille complète prendrait des heures ; Numba ramène ça à des minutes |

---

## Algorithmes — Edit Distance

| Décision | Raison |
|---|---|
| K-Means pour le codebook | Non supervisé ; entraîné sur les points de train indépendamment des étiquettes de classe |
| Edit distance normalisée par la longueur max de séquence | Empêche les séquences plus longues de toujours apparaître plus distantes |
| Compression (run-length encoding) comme HP | Supprime l'info de vitesse/durée des séquences symboliques ; ne conserve que la forme |
| Numba JIT sur arrays `uint8` | Les comparaisons d'entiers sont ~3× plus rapides que les comparaisons de caractères pour Numba |
| Deux lignes rolling dans la matrice DP | Réduit l'espace de O(N×M) à O(M) |

---

## Algorithmes — Three-Cents (3-Cent)

| Décision | Raison |
|---|---|
| **Pas de rotation** | L'orientation du geste est discriminative en 3D (swipe-gauche ≠ swipe-droit) ; la rotation effacerait cette distinction |
| **Scaling uniforme par longueur d'arc** (pas bounding-box) | Préserve les proportions du chemin ; le scaling bounding-box par axe distorserait la forme |
| 1-NN (pas de vote majoritaire) | 900 templates (un par geste d'entraînement) donnent déjà une couverture dense ; k>1 n'apporte que peu |
| Rééchantillonnage équidistant le long de l'arc | Représentation géométrique fidèle ; les points sont équidistants en distance, pas en temps |

> 💡 **Pourquoi 3-Cent bat $1 en 3D :** $1 fait une rotation vers un "angle indicatif" pour aligner les gestes — utile en 2D pour comparer un cercle dessiné dans n'importe quelle orientation. En 3D, un geste vers la gauche et vers la droite sont des classes **différentes** → NE PAS faire de rotation préserve cette distinction cruciale.

---

## Algorithmes — BiLSTM

| Décision | Raison |
|---|---|
| Architecture Bidirectionnelle | Capture les patterns depuis le début ET la fin du geste ; la fin d'un geste peut désambiguïser son début |
| BatchNorm après le LSTM | Stabilise l'entraînement sur petits datasets ; réduit la sensibilité au learning rate |
| Deux couches Dropout | Double régularisation (Srivastava et al. 2014) : sur les entrées LSTM et sur les sorties LSTM |
| Early stopping (patience=5) | Évite l'overfitting sur ~720–800 échantillons d'entraînement par fold sans tuner le nombre d'époques |
| Rééchantillonnage (pas padding+masking) | Plus simple ; équivalent à la normalisation de vitesse ; évite la complexité du masking |
| Label offset (`min(all_types)`) | Domaine 1 a des labels 1–10, Domaine 4 a 0–9 → offset pour compatibilité `to_categorical` |
| `tf.keras.backend.clear_session()` par fold | Libère la mémoire GPU ; évite que Keras construise un graphe de calcul croissant |

---

## Évaluation Statistique

| Décision | Raison |
|---|---|
| Test de Friedman (pas ANOVA) | Accuracies par fold bornées et non normales avec seulement 10 observations |
| Test pairé (exploite la structure des folds) | Toutes les méthodes évaluées sur les mêmes folds → le pairing supprime la variance inter-folds de la statistique de test |
| Correction post-hoc après Friedman global | Prévient le taux d'erreur de famille gonflé par 6 tests pairés indépendants |
| Les deux : Nemenyi ET Wilcoxon-Holm | Validation croisée des conclusions ; Wilcoxon-Holm est généralement plus puissant que Nemenyi |
| Comparer les méthodes à leur meilleure config HP | Comparaison équitable ; ce qui compte, c'est le plafond de performance de chaque méthode |
| Résultats user-independent comme comparaison principale | Requis par les directives du projet ; scénario d'évaluation réaliste |

---

## Décisions d'Implémentation (Numba & Performance)

| Décision | Raison |
|---|---|
| DTW et edit distance implémentés from scratch | Requis par le projet ; pas d'import de bibliothèques DTW/edit distance |
| Numba plutôt que Cython ou C extension | Plus simple à intégrer dans du code Python ; `@jit` décoré sur la fonction existante |
| Résultats sauvegardés en `.txt` + `_raw.csv` | `.txt` = lisible par un humain pour inspection ; `_raw.csv` = input du module d'évaluation statistique |
| Générateurs pour les splits CV | API propre ; pas de liste de tous les folds en mémoire |

---

## Connexions

- Résultats numériques → [[09_Results_and_Key_Findings]]
- DTW en détail → [[04_DTW_kNN]]
- Edit Distance en détail → [[05_Edit_Distance_kNN]]
- Three-Cents en détail → [[06_Three_Cent_Recognizer]]
- BiLSTM en détail → [[07_BiLSTM]]
