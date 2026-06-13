---
tags:
  - gesture-recognition
  - statistics
  - friedman
  - nemenyi
  - wilcoxon
  - hypothesis-testing
  - MLSMM2154
---

# Évaluation Statistique — Friedman, Nemenyi, Wilcoxon-Holm

> **Code :** `statistical_assessment.py`  
> **Input :** fichiers `_raw.csv` dans `results/` (un par méthode × domaine × mode CV)  
> **Output :** `results/statistical_tests.csv` + CD diagrams  
> **Lié à :** [[09_Results_and_Key_Findings]] · [[03_Cross_Validation_Strategy]] · [[00_Gesture_Recognition_MOC]]  
> **Paper :** [[Papers/Demsar2006_StatisticalComparisons]]

---

## Pourquoi des Tests Non-Paramétriques ?

Les accuracies par fold sont bornées dans [0, 1]. Avec seulement **10 folds**, le théorème central limite ne peut pas être invoqué pour supposer la normalité. Un ANOVA paramétrique serait invalide. Le **test de Friedman** est l'équivalent non-paramétrique approprié de l'ANOVA à mesures répétées.

---

## 1. Chargement des Données (`_load_best_fold_accuracies`)

Pour chaque fichier `_raw.csv` (une méthode) :
1. Détecter les colonnes d'hyperparamètres (colonnes avec plus d'une valeur unique)
2. Grouper par combinaison HP, calculer l'accuracy moyenne sur les folds
3. Identifier la meilleure configuration (accuracy moyenne maximale)
4. Extraire les accuracies **par fold** pour cette configuration

Le test statistique compare donc chaque méthode **à sa meilleure configuration** — pas à une configuration arbitraire.

### Structure du `_raw.csv`

```
fold_id | n_components | k | w | val_accuracy | accuracy
0       | no_pca       | 1 | None | 0.921     | 0.870
1       | no_pca       | 1 | None | 0.891     | 0.830
...
```

---

## 2. Matrice d'Accuracy (`_build_accuracy_matrix`)

Assemble un DataFrame **(10 folds × 4 méthodes)**. Chaque cellule = accuracy de test de cette méthode sur ce fold sous sa meilleure configuration HP.

```
         DTW    Edit    3-Cent  BiLSTM
Fold 0   0.87   0.74    0.98    0.89
Fold 1   0.83   0.71    0.96    0.85
...
Fold 9   0.81   0.76    0.94    0.82
Mean     0.827  0.746   0.963   0.857
```

---

## 3. Test de Friedman (`run_friedman`)

### Ce qu'il teste

$H_0$ : toutes les méthodes ont des performances équivalentes à travers les folds.

### Comment ça marche

1. **Classer** les 4 méthodes au sein de chaque fold (rang 1 = meilleure, rang 4 = moins bonne)
2. La statistique de Friedman $\chi^2_F$ résume l'accord entre ces classements par fold
3. Sous $H_0$, $\chi^2_F$ suit une distribution chi-deux avec $k - 1 = 3$ degrés de liberté

$$\chi^2_F = \frac{12N}{k(k+1)} \left[ \sum_j \bar{r}_j^2 - \frac{k(k+1)^2}{4} \right]$$

où $N$ = nombre de folds, $k$ = nombre de méthodes, $\bar{r}_j$ = rang moyen de la méthode $j$.

**Implémentation :** `scipy.stats.friedmanchisquare(*columns)` — chaque colonne = accuracies par fold d'une méthode.

### Rang Moyen

Calculé en classant dans chaque fold (accuracy plus haute = rang plus bas = meilleur) et en moyennant sur les folds. **Rang moyen plus bas = meilleure méthode.**

### Pourquoi Friedman plutôt que 6 t-tests ?

Faire 6 tests pairés à $\alpha = 0.05$ donne un taux d'erreur familial de ~26%. Friedman compare simultanément toutes les méthodes en contrôlant cette erreur. Il est aussi plus puissant car il exploite la **structure par paires** (même fold = même split train/test pour toutes les méthodes).

---

## 4. Test Post-Hoc de Nemenyi (`run_nemenyi`)

Appelé uniquement si le test de Friedman est significatif (p < α).

### Différence Critique (CD)

$$CD = q_\alpha \sqrt{\frac{k(k+1)}{6N}}$$

- $q_\alpha$ = valeur tabulée (Demšar 2006, Table 5) : pour $k = 4$ méthodes et $\alpha = 0.05$, $q_\alpha = 2.569$
- $k = 4$ méthodes, $N = 10$ folds
- Deux méthodes sont **significativement différentes** si $|\bar{r}_i - \bar{r}_j| \geq CD$

### Diagramme de Différence Critique (CD Diagram)

Style Demšar (2006) :
- Axe horizontal : rang 1 (meilleur) à droite, rang $k$ (moins bon) à gauche
- Chaque méthode = point à son rang moyen avec étiquette
- Bracket bleu CD en haut à gauche = distance critique
- **Barres noires épaisses** connectent les méthodes qui **ne sont PAS** significativement différentes (différence de rang < CD)

```
p-values : scikit_posthocs.posthoc_nemenyi_friedman(matrix.values)
```

---

## 5. Test de Wilcoxon + Holm-Bonferroni (`run_wilcoxon_holm`)

Test post-hoc alternatif. Plus puissant que Nemenyi car il utilise les valeurs réelles d'accuracy (pas seulement les rangs).

### Procédure

1. Faire tous les $\binom{4}{2} = 6$ tests de Wilcoxon pairés (test des signes avec rangs)
2. Trier les p-values : $p_{(1)} \leq p_{(2)} \leq \ldots \leq p_{(6)}$
3. **Correction Holm-Bonferroni** (step-down) :

$$p_{\text{adj},(i)} = \min\left(p_{(i)} \times (m - i + 1),\ 1.0\right)$$

où $m = 6$ (nombre de tests).

4. Rejeter $H_0$ pour la paire $i$ si $p_{\text{adj},(i)} < \alpha$

### Avantage sur Bonferroni

Holm-Bonferroni contrôle le taux d'erreur familial **sans supposer l'indépendance** entre les tests (contrairement à Bonferroni qui est plus conservateur). Moins de faux négatifs.

### Pourquoi Faire les Deux (Nemenyi ET Wilcoxon-Holm) ?

Si les deux tests concordent sur les paires significativement différentes → conclusion plus robuste. Nemenyi est le standard post-hoc pour Friedman ; Wilcoxon-Holm est généralement plus puissant. L'accord entre les deux renforce la confiance dans les résultats.

---

## 6. Interface en Ligne de Commande

```bash
python statistical_assessment.py                        # les deux domaines, CV indépendant
python statistical_assessment.py --domain 1 4           # spécifier les domaines
python statistical_assessment.py --cv independent       # uniquement user-independent
python statistical_assessment.py --cv both              # les deux modes CV
python statistical_assessment.py --posthoc both         # les deux tests post-hoc
python statistical_assessment.py --alpha 0.05           # niveau de significativité
python statistical_assessment.py --results_dir results  # dossier des CSVs
```

---

## 7. Fichiers de Sortie

| Fichier | Contenu |
|---|---|
| `results/statistical_tests.csv` | Toutes les statistiques de test et p-values (une ligne par test par paire) |
| `results/cd_diagram_domain1_independent.png` | CD diagram — Domaine 1, CV indépendant |
| `results/cd_diagram_domain4_independent.png` | CD diagram — Domaine 4, CV indépendant |

---

## 8. Résultats Statistiques (Domaine 1, User-Independent)

**Test de Friedman :** p < 0.001, $\alpha = 0.05$ → $H_0$ rejetée → les méthodes ne sont pas équivalentes.

**Conclusion Nemenyi + Wilcoxon-Holm :** 3-Cent est **statistiquement supérieur** à toutes les autres méthodes en mode user-independent. Les deux tests post-hoc concordent.

**Mode user-dependent :** Différences entre DTW, Edit-Distance et 3-Cent **non significatives** (tous > 98%). SeulBiLSTM est significativement inférieur.

---

## Connexions

- Résultats numériques par méthode → [[09_Results_and_Key_Findings]]
- Protocole CV qui génère les données per-fold → [[03_Cross_Validation_Strategy]]
- Paper de référence pour Friedman et CD diagram → [[Papers/Demsar2006_StatisticalComparisons]]
