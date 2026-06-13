---
tags:
  - gesture-recognition
  - data
  - dataset
  - data-loading
  - MLSMM2154
---

# Données et Chargement — Gesture Recognition

> **Dataset :** Huang et al. (2019) — 10 users × 10 classes × 10 reps = **1 000 séquences**  
> **Code :** `data/data_loading.py`  
> **Lié à :** [[03_Cross_Validation_Strategy]] · [[02_Preprocessing]] · [[00_Gesture_Recognition_MOC]]

---

## Structure Partagée des deux Domaines

Les deux domaines partagent la même structure :
- **10 sujets** (utilisateurs)
- **10 types de gestes** (classes)
- **10 répétitions** par (sujet, geste)
- **= 1 000 séquences** au total

Chaque séquence est une trajectoire 3D : une suite de points $(x, y, z)$ enregistrés dans le temps.

---

## Domaine 1 — Chiffres 0–9 tracés en 3D

**Contenu :** Chiffres 0 à 9 dessinés dans l'espace 3D (air gestures).

**Format :** Fichiers `.csv`, un par enregistrement.

**Convention de nom :** `subjectX-Y-Z.csv`
- `X` = numéro du sujet
- `Y` = type de geste (0–9)
- `Z` = numéro de répétition

**Contenu du fichier :** Colonnes `x, y, z, t` — la colonne `t` (temps) est **ignorée**.

**Pourquoi ignorer le temps ?** La tâche cible la forme spatiale du geste, pas sa vitesse. Les méthodes basées sur les templates (3-Cent) et le BiLSTM rééchantillonnent de toute façon. Le temps absolu n'apporte aucun signal utile et est ignoré universellement pour la cohérence.

**Chargement :** `load_data_domain_1()` dans `data/data_loading.py`.

```python
# Exemple de fichier : subject3-7-2.csv
# subject=3, gesture_type=7, repetition=2
x, y, z, t
0.12, 0.45, 0.33, 0.0
0.14, 0.47, 0.31, 0.1
...
```

**Détails techniques du loader :**
- `glob.glob() + sorted()` — ordre cohérent entre plateformes (glob ne garantit pas l'ordre alphabétique seul)
- `pd.to_numeric(..., errors='coerce').dropna()` — élimine silencieusement les lignes d'en-tête ou commentaires sans planter

---

## Domaine 4 — Figures Géométriques 3D

**Contenu :** 10 formes géométriques 3D dessinées dans l'air.

| Label | Forme |
|---|---|
| 0 | Cuboid (boîte rectangulaire) |
| 1 | Cylinder (cylindre) |
| 2 | Sphere (sphère) |
| 3 | Rectangular Pipe |
| 4 | Hemisphere (demi-sphère) |
| 5 | Cylinder Pipe |
| 6 | Pyramid (pyramide) |
| 7 | Tetrahedron (tétraèdre) |
| 8 | Cone (cône) |
| 9 | Toroid (tore) |

**Format :** Fichiers `.txt`, un par enregistrement.

**Structure :** Lignes 1–5 = métadonnées (domain ID, classe du geste, ID sujet). Les données commencent ligne 6. Seules les 3 premières colonnes $(x, y, z)$ sont lues — la colonne timestamp est ignorée.

**Inférence de la répétition :** Les noms de fichiers du Domaine 4 n'encodent PAS le numéro de répétition. Le loader l'infère en comptant le nombre de fois que la paire `(subject, gesture_type)` a déjà été vue, via un `defaultdict(int)`.

```python
# Principe du compteur de répétitions
from collections import defaultdict
rep_counter = defaultdict(int)
for file in sorted_files:
    subject, gesture_type = parse_metadata(file)
    repetition = rep_counter[(subject, gesture_type)]
    rep_counter[(subject, gesture_type)] += 1
```

**Chargement :** `load_data_domain_4()`.

---

## Représentation Interne Unifiée

Quel que soit le domaine source, chaque geste est stocké comme un dictionnaire Python :

```python
{
    "gesture_id":   int,          # ID séquentiel unique
    "subject":      int,          # Identifiant utilisateur (1–10)
    "gesture_type": int,          # Étiquette de classe (0–9 ou 1–10 selon domaine)
    "repetition":   int,          # Index de répétition au sein de (sujet, classe)
    "trajectory":   np.ndarray    # shape (n_timesteps, 3), dtype float64
}
```

Le Domaine 4 ajoute aussi `"gesture_name": str` (ex. : `"Cuboid"`).

**Pourquoi cette représentation ?**
- Interface uniforme pour tous les pipelines — aucun pipeline ne sait de quel domaine vient la donnée
- La trajectoire est une `np.ndarray` (n, 3) : compatible avec toutes les opérations numpy/numba directement
- `gesture_type` est le label de classe utilisé pour le CV et le scoring

---

## Statistiques des Données

| Domaine | # Sujets | # Classes | # Reps | Total | Longueur moy. trajectoire |
|---|---|---|---|---|---|
| Domain 1 (chiffres) | 10 | 10 | 10 | 1 000 | ~variable |
| Domain 4 (formes) | 10 | 10 | 10 | 1 000 | ~variable |

La longueur des trajectoires est **variable** entre les séquences — c'est pourquoi toutes les méthodes incluent soit une étape de DTW/edit-distance (qui gère les longueurs différentes), soit un rééchantillonnage à taille fixe (3-Cent, BiLSTM).

---

## Connexions

- Protocole de validation croisée → [[03_Cross_Validation_Strategy]]
- Normalisation et PCA → [[02_Preprocessing]]
- Utilisation dans les pipelines → [[04_DTW_kNN]] · [[05_Edit_Distance_kNN]] · [[06_Three_Cent_Recognizer]] · [[07_BiLSTM]]
- Paper du dataset → [[Papers/Huang2019]]
