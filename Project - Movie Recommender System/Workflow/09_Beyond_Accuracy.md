---
tags:
  - recommender-systems
  - workflow
  - diversity
  - novelty
  - coverage
  - ILD
  - MLSMM2156
---
# Beyond-Accuracy Metrics

RMSE measures prediction quality, but not recommendation usefulness. A model that recommends the same 10 blockbusters to every user has great RMSE on popular movies but is useless in practice.

Metric definitions and computation: [[Workflow/03_Split_and_Evaluation]] | Code: [[Codes/08_Ablation_and_Assessment_Code]]

---

## Full Results Table

| Model | CV RMSE | ILD | Novelty | Coverage | Hit Rate |
|---|---|---|---|---|---|
| User-based CF (Pearson k=80) | 0.8280 | 0.728 | **9.57** | 4.4% | 0.000 |
| Item-based CF (MSD k=40) | 0.8254 | 0.709 | 7.31 | 3.0% | 0.000 |
| NMF (f=10) | 0.8229 | **0.751** | 9.48 | 2.2% | 0.001 |
| SVD (tuned, f=100) | 0.7913 | 0.732 | 4.67 | 6.0% | 0.045 |
| SVD++ (f=150) | 0.7890 | 0.732 | 4.48 | **6.6%** | 0.035 |
| Content — TMDB full | 0.8878 | 0.443 | 6.18 | **9.2%** | — |
| Content — Genome + stats | 0.7346 | 0.653 | 5.48 | 8.3% | 0.059 |
| **Hybrid (SVD α=0.2 + Genome)** | **0.7308** | 0.663 | 4.98 | 8.0% | **0.064** |

---

## Intra-List Diversity (ILD)

**Higher = more genre-diverse recommendation lists.**

### Key Observations

**NMF has the highest ILD (0.751)** despite being the worst in its family for RMSE. Why?
- NMF's non-negativity constraint forces each of its 10 factors to encode a *distinct additive theme*
- Factors cannot cancel each other → top items from different factors come from different genre clusters
- Result: naturally diverse recommendation lists

**TMDB has the lowest ILD (0.443)** — the "auteur bubble":
- Director binary features create tight auteur clusters (Nolan fans get all Nolan films)
- Synopsis TF-IDF creates narrow narrative clusters
- Paradox: TMDB has the highest Coverage (9.2%) but lowest within-list diversity. Each user gets their own genre monoculture → the union of all lists is wide, but each individual list is narrow.

**Content genome ILD (0.653)** is the lowest non-TMDB value — confirms the **filter bubble effect**:
- Ridge on genome tags finds items that maximally match the user's tag profile → genre-homogeneous list
- A user who likes psychological thrillers gets a list of highly similar psychological thrillers

**The hybrid slightly recovers ILD (0.663)** over pure content: the 20% SVD arm introduces items that collective ratings favour regardless of content profile.

---

## Novelty

**Higher = more obscure recommendations (less popularity bias).**

Three tiers emerge:

### Tier 1: Obscure finders (Novelty > 6)
- User-based CF: **9.57** — neighbourhood aggregation propagates niche tastes
- NMF: **9.48** — parts-based factors rarely favour mainstream films
- Item-based CF: **7.31**

KNN methods surface niche items because the similarity computation naturally elevates movies that are similar *to the ones the specific user rated* — which are diverse across users.

### Tier 2: Balanced (Novelty 5–7)
- TMDB content: 6.18 (niche auteur preferences)
- Genome standalone: 5.48 (mild popularity bias from dense annotations on popular films)
- Hybrid: 4.98

### Tier 3: Popularity-biased (Novelty < 5)
- SVD: 4.67
- SVD++: **4.48** (lowest novelty overall)

SVD++ amplifies popularity bias: popular movies appear in every user's implicit interaction history, reinforcing their gravity in the latent space. The `yⱼ` implicit vectors collectively pull predictions toward frequently-seen items.

---

## Hit Rate (LOO, top-40)

**Higher = model correctly retrieves the held-out item in top-40.**

| Score | Interpretation |
|---|---|
| 0.000 | CF families — neighbourhood aggregation surfaces popular consensus, not individual items |
| 0.001 | NMF — niche factors rarely coincide with typical user choices |
| 0.035–0.045 | SVD/SVD++ — item bias bⁱ boosts frequently-rated films |
| 0.059 | Genome + stats — Ridge on perceptual tags directly maps to ratings |
| **0.064** | **Hybrid** — SVD correction further lifts collaborative favourites |

The content models win on Hit Rate because Ridge directly maps a movie's perceptual tag profile to a predicted rating — if the user's weight vector aligns with the held-out item's tags, the item rises to the top of the list.

---

## The Accuracy–Diversity Trade-Off

No model dominates all dimensions:

```
                    RMSE ↓           ILD ↑        Novelty ↑
─────────────────────────────────────────────────────────────
CF (KNN)           0.826–0.828    0.71–0.75      7.3–9.6 ★
SVD/SVD++          0.789–0.791    0.73           4.5–4.7
NMF                0.823          0.75 ★         9.5 ★
Genome + stats     0.735          0.65           5.5
Hybrid             0.731 ★        0.66           5.0
TMDB full          0.888          0.44 ↓         6.2
```

**The hybrid model achieves the best accuracy AND reasonable diversity:**
- Best RMSE: 0.7308
- Coverage: 8.0% (second highest after TMDB)
- ILD: slight gain over pure content (SVD arm broadens recommendations)
- Novelty: 4.98 (well above pure SVD's 4.67 — content arm anchors away from pure popularity bias)

---

> [!WARNING]+ ❓ Assessor Question — What is the filter bubble effect and how does it appear in the data?
> A filter bubble occurs when a recommender shows users only content similar to what they've already consumed, reinforcing preferences without exposing them to new genres or styles. In our data: genome + stats produces ILD = 0.653 — the lowest of all non-TMDB models. Users who like one genre get lists packed with that genre. The hybrid partially breaks this (ILD = 0.663) because the SVD arm introduces items that many users collectively like, some of which will be in different genres.

> [!WARNING]+ ❓ Assessor Question — Why do KNN models have Hit Rate = 0.000?
> KNN predicts by aggregating the ratings of similar users/items. The result is a "consensus" prediction — what a typical user with this profile would rate. The top-40 list under KNN is therefore dominated by popular, well-reviewed films that *similar users* liked in general. The single held-out item for a specific user is often a niche personal preference that doesn't align with the popular consensus. KNN never "discovers" personal niche items. Content models do better because they learn the user's specific taste profile, not just their average neighbourhood.

> [!WARNING]+ ❓ Assessor Question — If TMDB has the highest coverage (9.2%), why is it the worst-performing content model?
> High coverage means many different movies appear across recommendation lists. TMDB achieves this because director-based features create distinct per-user niches: a Nolan fan gets Nolan films, a Kubrick fan gets Kubrick films — different users get different clusters, maximising catalogue coverage. But the underlying RMSE is poor (0.888) because director/synopsis features don't actually predict ratings well. Coverage is a property of the recommendation *diversity across users*, not of the recommendation *accuracy for any individual user*.

---

---

## 📚 Références Académiques

- **Métriques beyond-accuracy — cadre théorique complet :** [[Papers/Castells2015_NoveltyDiversity]] — ILD, MIUF, Unexpectedness, Aggregate Diversity, algorithme MMR, bulle de filtre, longue traîne
- **Vue d'ensemble des métriques d'évaluation RecSys :** [[Papers/Chen2017_PerformanceEvaluation]] — taxonomy ML/IR/HCI/Engineering, formules MAE/RMSE/Precision/Recall/nDCG/Coverage
- **Impact du découpage des données :** [[Papers/Meng2020_DataSplitting]] — Leave One Last vs Temporal Global, Kendall τ, variable confondante

---

**See also:** [[Workflow/08_Results_and_Decision_Making]] | [[Workflow/03_Split_and_Evaluation]] | [[Workflow/07_Statistical_Assessment]]
