---
title: "Gesture Recognition — Project MOC"
course: MLSMM2154
tags:
  - MOC
  - gesture-recognition
  - supervised-learning
  - sequence-classification
  - MLSMM2154
  - project
created: 2026-05-22
updated: 2026-06-13
---

# Gesture Recognition MLSMM2154 — Knowledge Base

> **Cours :** MLSMM2154 — Artificial Intelligence (UCLouvain Mons)  
> **Professeur :** Marco Saerens · **Assistants :** Alexis Airson, Diego Eloi, Nicolas Szelagowski  
> **Dataset :** [[Papers/Huang2019|Huang et al. (2019)]] — 10 users × 10 classes × 10 reps = **1 000 séquences**  
> **Dépôt code :** [Matteo-glz/MLSMM2154_Artificial-Intelligence_gesture_recognition](https://github.com/Matteo-glz/MLSMM2154_Artificial-Intelligence_gesture_recognition)  
> **Master knowledge graph :** [[00_AI_Knowledge_Graph_MOC|AI Knowledge Graph MOC]]

---

## 🏆 Meilleur Résultat

**Three-Cents — 96.3% ± 4.6 % (User-Independent, Domaine 1)**  
Statistiquement supérieur à toutes les autres méthodes (Friedman p < 0.001).  
→ [[09_Results_and_Key_Findings|Résultats complets]]

---

## 📁 Index des Notes du Projet

### 🔬 Pipelines — Les 4 Méthodes

| Note | Rôle | Approche | Résultat UI Dom.1 |
|---|---|---|---|
| [[04_DTW_kNN]] | Baseline | DTW + k-NN (elastic alignment) | 82.7 ± 14.6 % |
| [[05_Edit_Distance_kNN]] | Baseline | K-Means → chaîne symbolique → Levenshtein + k-NN | 74.6 ± 20.9 % |
| [[06_Three_Cent_Recognizer]] | SOTA template | Rééchantillonnage → normalisation géométrique → 1-NN | **96.3 ± 4.6 %** |
| [[07_BiLSTM]] | SOTA deep learning | Bidirectional LSTM → softmax | 85.7 ± 11.7 % |

### ⚙️ Infrastructure & Protocole

| Note | Contenu |
|---|---|
| [[01_Data_and_Datasets]] | Structure des 2 domaines, data loading, représentation interne unifiée |
| [[02_Preprocessing]] | Normalisation Z-score + PCA par classe — protocole anti-leakage |
| [[03_Cross_Validation_Strategy]] | User-Independent vs User-Dependent, inner val split, Phase 1 & 2 |
| [[08_Statistical_Assessment]] | Friedman, Nemenyi, Wilcoxon-Holm, CD diagram — pourquoi non-paramétrique |
| [[09_Results_and_Key_Findings]] | Tableau complet des résultats, 5 observations clés, analyse des variances |
| [[10_Design_Decisions]] | Toutes les décisions de conception avec justification technique |

### 📋 Autres Documents

| Note | Contenu |
|---|---|
| [[Project Assignment]] | Énoncé officiel du projet |
| [[Présentation]] | Notes de présentation orale |

---

## 🔄 Pipeline Complet (Vue d'Ensemble)

```
Données brutes (x, y, z) — Domaines 1 & 4
         │
         ▼  [01_Data_and_Datasets]
   data_loading.py → représentation unifiée {trajectory, subject, gesture_type, ...}
         │
         ▼  [03_Cross_Validation_Strategy]
   user_independent_cv() ou user_dependent_cv() → 10 folds
         │
         ▼  [02_Preprocessing] — appliqué à chaque fold
   fit_normalizer(train) → apply_normalizer(train, test)
   [optionnel] fit_pca_per_gesture(train) → apply_pca_per_gesture(...)
         │
         ├─────────────────────────────────────────────┐
         │  Phase 1 : inner_val_split → grille HP      │
         │  Phase 2 : meilleur HP fixé → éval. test   │
         └─────────────────────────────────────────────┘
         │
         ▼  [04 / 05 / 06 / 07]
   DTW k-NN  |  Edit-Distance k-NN  |  3-Cent 1-NN  |  BiLSTM
         │
         ▼  [08_Statistical_Assessment]
   Friedman + Nemenyi + Wilcoxon-Holm → CD diagram
         │
         ▼  [09_Results_and_Key_Findings]
   Tableau final + conclusions
```

---

## 📐 Hyperparamètres par Méthode

### [[04_DTW_kNN]]
| HP | Valeurs testées |
|---|---|
| `n_components` | `no_pca`, 2, 3 |
| `k` (voisins) | 1, 3, 5, 7 |
| `w` (Sakoe-Chiba) | None, 10, 20 |

### [[05_Edit_Distance_kNN]]
| HP | Valeurs testées |
|---|---|
| `n_components` | `no_pca`, 2, 3 |
| `n_clusters` (alphabet) | 15, 20, 25, 30, 35, 40 |
| `k` (voisins) | 1, 3, 5, 7 |
| `compression` | True, False |

### [[06_Three_Cent_Recognizer]]
| HP | Valeurs testées |
|---|---|
| `n_components` | `no_pca`, 2, 3 |
| `n_points` (rééchantillonnage) | 16, 32, 64 |

### [[07_BiLSTM]]
| HP | Valeurs testées |
|---|---|
| `target_length` | 16, 32, 64 |
| `n_units` (LSTM hidden) | 16, 32, 64 |
| `dropout_rate` | 0.1, 0.2, 0.3, 0.5 |

---

## 🔗 Concepts et Papers Liés

### Algorithmes Clés
- [[Concepts/Dynamic_Programming|Dynamic Programming]] — backbone commun DTW, Edit Distance, LCS
- [[Concepts/kNN_Classifier|k-NN Classifier]] — utilisé par DTW, Edit Distance
- [[Concepts/Vector_Quantization|Vector Quantization]] — K-Means pour l'alphabet symbolique (Edit Distance)
- [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]] — alignement rotation optimal ($1/$3)
- [[Concepts/Cross_Validation|Cross-Validation Protocols]] — LOUO (user-independent), LOSO (user-dependent)

### Deep Learning (BiLSTM)
- [[Concepts/Backpropagation|Backpropagation]] · [[Concepts/Neural_Networks_MLP|Neural Networks]]
- [[Papers/Hochreiter1997|Hochreiter & Schmidhuber (1997)]] — LSTM
- [[Papers/Schuster1997|Schuster & Paliwal (1997)]] — BiLSTM
- [[Papers/Srivastava2014_Dropout|Srivastava et al. (2014)]] — Dropout
- [[Papers/Ioffe2015_BatchNorm|Ioffe & Szegedy (2015)]] — Batch Normalization

### Papers Fondamentaux
- [[Papers/Sakoe1978|Sakoe & Chiba (1978)]] — DTW
- [[Papers/Levenshtein1966|Levenshtein (1966)]] · [[Papers/Wagner1974|Wagner & Fischer (1974)]] — Edit Distance
- [[Papers/Kratz2010|Kratz & Rohs (2010)]] · [[Papers/Wobbrock2007|Wobbrock et al. (2007)]] — $3 / $1 Recognizer
- [[Papers/Demsar2006_StatisticalComparisons|Demšar (2006)]] — Friedman + CD diagram

---

## ✅ Checklist Rapport

- [x] Introduction (problème, dataset, 4 méthodes)
- [x] Données : Domain 1 et Domain 4 → [[01_Data_and_Datasets]]
- [x] Prétraitement : normalisation, PCA → [[02_Preprocessing]]
- [x] Protocole CV : UI vs UD, Phase 1 & 2 → [[03_Cross_Validation_Strategy]]
- [x] Théorie DTW avec équations → [[04_DTW_kNN]]
- [x] Théorie Edit Distance + VQ → [[05_Edit_Distance_kNN]]
- [x] Théorie 3-Cent ($3) → [[06_Three_Cent_Recognizer]]
- [x] Théorie BiLSTM → [[07_BiLSTM]]
- [x] Tableau résultats (mean ± std × {UI, UD} × {Dom1, Dom4}) → [[09_Results_and_Key_Findings]]
- [x] Tests statistiques (Friedman, Nemenyi, Wilcoxon-Holm) → [[08_Statistical_Assessment]]
- [x] Décisions de conception → [[10_Design_Decisions]]
- [ ] Diagramme CD (généré par `statistical_assessment.py`)
- [ ] Matrice de confusion (meilleure méthode)

---

## 📚 Références

Entrées BibTeX : `gesture_recognition.bib`  
Index complet des papers : [[Papers/00_Papers_MOC|Papers MOC]]  
Import Zotero : **File → Import → gesture_recognition.bib**
