---
tags:
  - recommender-systems
  - MOC
  - MLSMM2156
  - movielens
  - index
---
# RecSys MLSMM2156 — Knowledge Base

Welcome to the knowledge base for the **MovieLens Recommender System** project (MLSMM2156 Hackathon).

---

## 📁 Folders

### [[Workflow/01_Project_Overview|🔁 Workflow]]
End-to-end pipeline, data, splitting, evaluation, tuning, and decision-making.

| Note | Topic |
|---|---|
| [[Workflow/01_Project_Overview]] | Dataset at a glance, goals, model families |
| [[Workflow/02_Data]] | MovieLens dataset, features, rating distribution |
| [[Workflow/03_Split_and_Evaluation]] | K-Fold CV, LOO, train/test protocol |
| [[Workflow/04_Feature_Engineering]] | All content feature methods explained |
| [[Workflow/05_Hyperparameter_Tuning]] | Strategy, grids, results per family |
| [[Workflow/06_Ablation_Study]] | How the ablation pipeline works |
| [[Workflow/07_Statistical_Assessment]] | Friedman test, Nemenyi, Wilcoxon |
| [[Workflow/08_Results_and_Decision_Making]] | RMSE tables, best models per family |
| [[Workflow/09_Beyond_Accuracy]] | ILD, Novelty, Coverage, Hit Rate |
| [[Workflow/10_Content_Based_Theory]] | Content-based filtering — full theory (TF-IDF, profiling, filtering) |
| [[Workflow/11_Latent_Factor_Theory]] | Latent factor models — full theory (MF, SVD, Netflix Prize) |
| [[Workflow/12_Rating_Bias_and_Statistics]] | Mean-centering, Bayesian shrinkage, similarity metrics |
| [[Workflow/14_Review_and_Gaps]] | Project review: gaps, open questions, random seed, TODOs |
| [[Workflow/15_Evaluator_vs_Ablation]] | Why LOO is expensive; evaluator vs ablation differences |

---

### [[Codes/01_Constants_and_Loaders|💻 Codes]]
Every Python file in `recsys/` explained from scratch.

| Note | File |
|---|---|
| [[Codes/01_Constants_and_Loaders]] | `constants.py` + `loaders.py` |
| [[Codes/02_Feature_Engineering_Code]] | `build_content_features()` factory |
| [[Codes/03_Models_Baselines]] | `ModelBaseline1–4` |
| [[Codes/04_Models_Collaborative]] | `UserBased`, `ItemBased` |
| [[Codes/05_Models_Latent_Factor]] | `SVDModel`, `SVDppModel`, `NMFModel` |
| [[Codes/06_Models_Content_Based]] | `ContentBased`, `ContentBasedWithStats`, `ContentBasedPlus` |
| [[Codes/07_Models_Hybrid]] | `HybridSVD` |
| [[Codes/08_Ablation_and_Assessment_Code]] | `ablation_study.py` + `statistical_assessment.py` |
| [[Codes/09_Configs]] | `EvalConfig`, `WebappConfig` |

---

### [[Webapp/01_Webapp_Overview|🌐 Webapp]]
Flask server and recommendation serving pipeline.

| Note | Topic |
|---|---|
| [[Webapp/01_Webapp_Overview]] | Architecture, startup, model selection |
| [[Webapp/02_Server_Endpoints]] | `/recommend`, `/similar`, cold/warm start cascade |
| [[Webapp/03_Cascade_Architecture]] | Phase 0/1/2 cascade, ContentBasedWithStats, Ridge justification |

---

## 🏆 Best Results at a Glance

| Family | Best Config | CV RMSE |
|---|---|---|
| Item-based CF | MSD, k=40 | 0.8254 ± 0.0011 |
| User-based CF | Pearson, k=80 | 0.8280 ± 0.0023 |
| Latent Factor (SVD) | f=100, λ=0.05, η=0.01 | 0.7913 ± 0.0015 |
| Latent Factor (SVD++) | f=150, λ=0.05, η=0.01 | 0.7890 ± 0.0024 |
| Content-Based | Genome full + stats, Ridge α=20 | 0.7346 ± 0.0015 |
| **Hybrid** | **SVD (α=0.2) + Genome Ridge** | **0.7308 ± 0.0015** |
