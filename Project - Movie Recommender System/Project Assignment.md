---
title: "MLSMM2156 — Movie Recommender System: Project Assignment"
course: "MLSMM2156 — Recommender Systems"
tags:
  - project
  - recommender-systems
  - collaborative-filtering
  - content-based
  - latent-factor
  - movielens
  - streamlit
  - assignment
created: 2026-05-25
deadline: TBD (see Moodle)
authors:
  - GALIZIA Matteo
  - MINET Bastian
  - SIMOENS Mathias
---

# MLSMM2156 — Movie Recommender System

> **Course:** MLSMM2156 — Recommender Systems
> **Goal:** Build a complete movie recommender system combining user-based, content-based, and latent factor models, evaluated on the MovieLens dataset.
> **Deliverable:** Report (15 p. max) + Python code + live demo + oral defence (40 min)

---

## 📋 Project Overview

Build a **top-n movie recommender system** by combining all components acquired through continuous evaluation. The system must be original per group — different functionalities and models are expected.

**Dataset:** MovieLens *small* version and/or *hackathon* version. Additional external data sources are permitted.

---

## 🏗️ System Architecture

The expected architecture (from project diagram) separates concerns into four modules:

| Module | File | Role |
|---|---|---|
| **Analytics** | `analytics.py` | Data exploration and analysis |
| **WebApp** | `content.py` | Orchestrates the full pipeline |
| **Recommender** | `recommender.py` | Core recommendation logic |
| **Models** | `models.py` | All recommendation models |
| **Evaluator** | `evaluator.py` | Offline evaluation pipeline |
| Analytics UI | `analytics.ipynb` | Jupyter-based analytics dashboard |
| Recommender UI | `app.py` | Streamlit user-facing web app |
| Config | `configs.py` | Project configurations |
| Constants | `constants.py` | MovieLens properties |

Data is split into four stores: **Content** (item attributes), **Evidence** (ratings), **Recs** (trained models), **Evaluations** (evaluation reports).

---

## ✅ Mandatory Requirements

### Models (minimum)
- **(a) User-based model** — collaborative filtering with neighbourhood approach
  - Must include **at least one custom similarity metric** not available in the `surprise` package
  - Compare against other user-based models
- **(b) Content-based model** — item attribute-driven recommendations
- **(c) Latent factor model** — matrix factorisation (e.g., SVD, NMF, ALS)

### System features
- [ ] **Top-n recommendations** for any user
- [ ] **Web app** (Streamlit or equivalent) — `app.py`
- [ ] **Explanation** of recommendations shown to the end user
- [ ] **Performance comparison** across models on well-chosen metrics
- [ ] **Diversity metric** included in evaluation

---

## 📊 Evaluation Metrics (to choose and justify)

At minimum, cover complementary dimensions:

| Dimension | Example metrics |
|---|---|
| **Accuracy** | RMSE, MAE, Precision@k, Recall@k, NDCG@k |
| **Ranking** | MRR, MAP |
| **Diversity** | Intra-list diversity, coverage, novelty |
| **Beyond accuracy** | Serendipity, unexpectedness |

> Every metric choice must be **justified** in the Experiments section.

---

## 📝 Report Structure (15 pages max)

| Section | Length | Content |
|---|---|---|
| **1. Introduction** | 1 page max | Short project introduction |
| **2. Live demo** | 1 URL | YouTube/other link — explain UI, choices, interactions |
| **3. Experiments** | Main body | Model comparison, metric discussion, hyperparameter justification |
| **4. Discussion & Conclusion** | 1 page max | Strengths and weaknesses |

**Submission:** `MLSMM2156_project_groupi.zip` containing `report.pdf` + Python scripts folder (no data files).
Also submit slides (PDF) on Moodle.

---

## 🎯 Oral Defence (40 min)

| Part | Duration |
|---|---|
| Live demo (preferred over video) | 5 min |
| Performance results presentation | 5 min |
| Q&A — theoretical + project questions | 30 min |

> Questions can be addressed to the group **or to a specific member**. Individual understanding will be assessed.

---

## 📐 Evaluation Criteria (/20)

| Component | Points | Criteria |
|---|---|---|
| Coding assignments (×4) | /4 | 0 or 1 per assignment — correctly completed |
| Project + oral defence | /16 | See competencies below |

### Four competencies

**C1-EXPL** — *Understanding & creativity*
Clear understanding of recommender system concepts. Workflow creatively incorporates different models. Strengths and weaknesses highlighted throughout.

**C2-APPL** — *Code quality*
High quality, well-documented code. Clear instructions for each component. Runs properly. Extra points for Streamlit/web app. Efficient implementation (offline/online considerations, leverages existing packages).

**C3-DISC** — *Experimental rigour*
Every choice justified by experimental results. Charts and graphs support decisions. Metrics are complementary and appropriate.

**C4-COMM** — *Communication*
Well-written report following submission guidelines. Live demo clearly illustrates how the project was built.

> **Creativity is strongly encouraged and appropriately rewarded.**

---

## ⚠️ AI Disclosure

Use of generative AI is **permitted** but must be explicitly disclosed in the report, in compliance with [UCLouvain AI guidelines](https://www.uclouvain.be/fr/ia/documents). The oral presentation plays a crucial role precisely because of increasing AI use — individual understanding will be assessed directly.

---

## 🗓️ Checklist

- [ ] Three model types implemented (user-based, content-based, latent factor)
- [ ] Custom similarity metric for user-based model
- [ ] Streamlit web app (`app.py`) running
- [ ] Recommendation explanations for end users
- [ ] Diversity metric included in evaluation
- [ ] All model choices justified by experimental results
- [ ] Live demo recorded (YouTube link)
- [ ] Report ≤ 15 pages, in English
- [ ] `MLSMM2156_project_groupi.zip` ready (no data files)
- [ ] Slides (PDF) ready
- [ ] AI use disclosed in report

---

## 🔗 Connections

- Dataset → [[Papers/Huang2019|Huang et al. (2019)]] *(MovieLens-style gesture data — different domain but same evaluation philosophy)*
- Evaluation framework → [[Workflows/Model_Evaluation|Model Evaluation]]
- Cross-validation → [[Concepts/Cross_Validation|Cross-Validation Protocol]]
- Latent factor models → [[Concepts/Dimensionality_Reduction|Dimensionality Reduction (SVD/PCA)]]
- Similarity metrics → [[Concepts/kNN_Classifier|k-NN Classifier]]
- Diversity & beyond-accuracy metrics → [[Concepts/Unsupervised_Learning|Unsupervised Learning]]
