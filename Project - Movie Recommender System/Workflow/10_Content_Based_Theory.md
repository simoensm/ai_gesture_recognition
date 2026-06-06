---
tags: [recommender-systems, content-based, NLP, profiling, machine-learning]
---
# Content-Based Filtering — Theory

**Course:** MLSMM2156 — Recommender Systems
**Related concepts:** [[Concepts/Content_Based_Filtering]] | [[Concepts/TF-IDF]] | [[Codes/06_Models_Content_Based]] | [[Workflow/04_Feature_Engineering]]

---

## 📍 Where We Are

```
Recommender System Models
│
├── [[Collaborative Filtering]]   ← memory-based & model-based
├── Content-based                 ← YOU ARE HERE
├── [[Demographic]]
├── [[Knowledge-based]]
└── [[Context-based]]
```

Content-based sits alongside [[Collaborative Filtering]] but works on **fundamentally different input data**: instead of leveraging the ratings of _other_ users, it relies on the **attributes of the items** and the **ratings of the target user alone**.

---

## 1. Motivation

### 1.1 The Intuition

Imagine a conversation:

> **Bob:** I recently watched _Black Mirror_. I loved it. **Ann:** So you like technology that goes wrong? **Bob:** Exactly! **Ann:** Then you'll probably enjoy _Ex Machina_ and _Terminator_.

Ann is not asking _"who else liked Black Mirror?"_ — she is extracting **features** from the content (dystopian tech, dark tone) and matching them to other items that share those features. This is precisely what a content-based recommender does.

### 1.2 The Cold-Start Problem for Items

Consider a rating matrix $\mathbf{R}$:

$$ \mathbf{R} = \begin{pmatrix} 3 & & 8 & \mathbf{?} & 9 & 6 & & 1 \ 2 & & 4 & \mathbf{?} & 4 & & 10 & 8 \ 10 & 9 & & \mathbf{?} & 4 & 6 & 3 & 2 \ & 10 & & \mathbf{?} & & & 4 & \ & 6 & 6 & \mathbf{?} & 8 & & 5 & 2 \ & 8 & 10 & \mathbf{?} & & & & \ \end{pmatrix} $$

When an item column is entirely empty (new or unpopular item), **[[Collaborative Filtering]] cannot work** — there is nothing to compare against. Content-based approaches have no such limitation because they use item attributes, not user–item co-occurrence.

> 💡 **Key use case:** Content-based filtering is particularly powerful for **new** and **unpopular** items.

---

## 2. Basics and Components

### 2.1 High-Level Architecture

A content-based recommender has three main components (Ricci et al., 2011):

```
Information Source (raw item descriptions)
        │
        ▼
┌──────────────────┐
│  CONTENT ANALYZER│  → transforms raw descriptions into structured representations
└──────────────────┘
        │  (item feature vectors)
        ▼
┌──────────────────┐
│  PROFILE LEARNER │  ← user feedback (ratings)
└──────────────────┘
        │  (user profile)
        ▼
┌──────────────────────┐
│  FILTERING COMPONENT │  → scores & ranks unseen items for active user
└──────────────────────┘
        │
        ▼
   List of Recommendations
```

These components are also called **attributes**, **features**, or **descriptors** in the literature.

### 2.2 Input Data

|Input|Description|
|---|---|
|User–item ratings|Only for the **target user** (not all users)|
|Item attributes|Structured or textual metadata|

> **Intuition:** If two books are both thrillers by the same author, and you liked one, you will likely enjoy the other.

---

## 3. The Content Analyzer

The content analyzer **converts raw item descriptions into structured numerical representations** that can be processed by a model.

### 3.1 Example: Black Mirror

Raw IMDB page → structured item record:

|Field|Value|
|---|---|
|`title`|Black Mirror (2011–2019)|
|`description`|An anthology series exploring a twisted, high-tech multiverse where humanity's greatest innovations and darkest instincts collide.|
|`length`|52 min|
|`genre`|drama, mystery, sci-fi|
|`actors`|John Hamm, Lenora Richlow|
|`tags`|technology, ladygaga|
|`imdb_rank`|212|

### 3.2 Facts vs. Tags

||**Facts**|**Tags**|
|---|---|---|
|Source|Objective metadata|Community-generated|
|Nature|Cannot be disputed|Subjective, varied vocabulary|
|Examples|genre, length, year|"technology", "mind-bending"|
|Usefulness|Core structure|Can **uplift** recommendations|

### 3.3 Attribute Types

Item data falls into four types — and most of it is textual:

|Type|Example field|Example value|
|---|---|---|
|**Nominal**|Genres|drama, mystery, sci-fi|
|**Ordinal**|IMDB Rank|212|
|**Interval**|Length (min)|52|
|**Textual**|Title, Tags, Description|"Black Mirror…"|

> ⚠️ Data is **dominated by textual types** — but models require numbers. Hence the need for the encoding and [[#3.5 Text Mining Pipeline]] steps below.

### 3.4 Encoding Nominal Variables

Nominal variables (e.g., genres) are converted to binary indicator columns via **one-hot encoding**:

$$ \text{genres} = {\text{drama, mystery, sci-fi}} \quad\longrightarrow\quad \begin{array}{|c|c|c|c|} \hline \texttt{is_drama} & \texttt{is_horror} & \texttt{is_mystery} & \texttt{is_thriller} \ \hline 1 & 0 & 1 & 0 \ \hline \end{array} $$

Other encoding techniques include: Ordinal Coding, Sum Coding, Helmert Coding, Polynomial Coding, Backward Difference Coding, Binary Coding.

### 3.5 Extracting Structure from Text: Regular Expressions

Naïve character slicing is **fragile**:

```python
# Works for: 'Black Mirror (2011-2019)'
title[-5:-1]   # → '2019' ✅
title[-10:-6]  # → '2011' ✅

# Breaks for: 'Big Bang Theory, The (2007-)'
title[-5:-1]   # → '007-' ❌
title[-10:-6]  # → 'he (2' ❌
```

Other edge cases that break fixed slicing:

|Title|Problem|
|---|---|
|`'Black Mirror (2011-2019)'`|Two years|
|`'Big Bang Theory, The (2007-)'`|No end year|
|`'Stranger Things'`|No year at all|
|`'Hostal: Part III (2011) '`|Trailing space|

**Solution: Regular Expressions** (`import re` in Python; playground at [regexr.com](https://regexr.com/))

Example — extract start year from any title:

```
\((\d{4})
```

- `\(` → literal opening parenthesis
- `(\d{4})` → capture group: exactly 4 digits

For email extraction, a practical regex is:

```
[\w\-\.]+@([\w\-]+\.)+[\w\-]{2,4}
```

> A fully RFC-compliant email regex spans ~800 characters — a good reminder that regex complexity scales with edge cases.

### 3.6 Text Mining Pipeline

Converting free text (e.g., movie descriptions) to numerical weights follows this pipeline:

```
Raw Text
   │
   ▼  Tokenization
["High-tech", "multiverse", "where", "humans", "greatest", "innovations", …]
   │
   ▼  Stop-word Removal
["High-tech", "multiverse", "humans", "greatest", "innovations", "darkest", …]
   │
   ▼  Stemming / Lemmatization
["hightech", "multiverse", "human", "great", "innov", "dark", "instinct", "collid"]
   │
   ▼  Frequency Filtering  (remove very rare / very common tokens)
["hightech", "human", "great", "dark"]
   │
   ▼  Vectorization
{"hightech": 0.88, "human": 0.62, "great": 0.20, "dark": 0.90}
```

### 3.7 Vectorization Options

#### Option 1 — TF-IDF (Term Frequency–Inverse Document Frequency)

Each term $w$ in document $d$ in corpus $D$ gets a weight:

$$ \text{TF-IDF}(w, d, D) = \underbrace{\frac{f_{w,d}}{\sum_{w'} f_{w',d}}}_{\text{TF}} \times \underbrace{\log\frac{|D|}{|{d' \in D : w \in d'}|}}_{\text{IDF}} $$

- **TF** rewards terms that appear often _in this document_
- **IDF** penalises terms that appear in _almost every document_ (uninformative)

Result: each item becomes a **sparse vector** in a high-dimensional word space.

#### Option 2 — Topic Modelling with LDA

**Latent Dirichlet Allocation** reduces text to a small set of latent topics.

|Concept|Meaning|
|---|---|
|_Latent_|Topics are hidden; discovered by the algorithm|
|_Dirichlet_|The model is a distribution over distributions|
|_Allocation_|Every word is allocated to a topic|

```
Document: "Spiderman is home with his laptop to eat breakfast with a fork."
   │
   ├──► Topic: Superhero  (weight: 0.60) → words: Flying, Strong, Superhero
   ├──► Topic: Computer Science (weight: 0.30) → words: computer, laptop
   └──► Topic: Food (weight: 0.10) → words: breakfast, fork
```

**Advantages of LDA:**

- **Dimensionality reduction**: thousands of words → tens of topics
- **Explainability**: "Because you like Sci-Fi thrillers…"
- Each item is represented as a **dense vector** of topic weights

> 🐍 Python library: **Gensim** — `from gensim.models import LdaModel`

### 3.8 Normalisation

Before using item vectors, always normalise interval/continuous attributes so that no single feature dominates by scale:

$$ \text{year}_{\text{norm}} = \frac{\text{year} - \text{year}_{\min}}{\text{year}_{\max} - \text{year}_{\min}} $$

Example:

|Item|`is_drama`|`is_sci`|`year` (raw)|`year` (norm)|`topic_0`|
|---|---|---|---|---|---|
|Black Mirror|1|1|2011|0.50|1|
|Ex Machina|0|1|2014|0.80|0|

---

## 4. Profiling

The **Profile Learner** uses the item feature vectors + the user's known ratings to build a **user profile** — a representation of the user's tastes in the same feature space as the items.

### 4.1 Overall ML Workflow

```
Content Analyzer output (item feature vectors)
        │
        ▼
Feature Engineering   →   Feature Scaling   →   Feature Selection & Weighting
        │
        ▼
   Modelling (classifier or regressor)
        │
        ▼
   Evaluation & Tuning
        │
        ▼
Scoring & Filtering  →  Recommendations
```

### 4.2 Unsupervised Feature Selection (in the Content Analyzer)

The text mining pipeline already performs implicit, **unsupervised** feature selection:

- **Frequency Filtering** removes tokens that are too rare or too common
- **TF-IDF weighting** suppresses uninformative terms automatically

No user signal is needed here.

### 4.3 Supervised Feature Selection (using Ratings)

When user ratings are available, we can ask: _does this feature actually predict whether the user likes an item?_

#### For unary / binary ratings — $\chi^2$ Statistic

**Example setup:**

- Netflix library: 1000 movies
- User watched 10% → 100 movies watched
- 20% of movies are dramas → 200 drama movies

**Expected table** (if "drama" has _no_ impact on the user's choice):

||Term occurs|Term does not occur|
|---|---|---|
|User watched item|$1000 \times 0.1 \times 0.2 = 20$|$1000 \times 0.1 \times 0.8 = 80$|
|User did not watch|$1000 \times 0.9 \times 0.2 = 180$|$1000 \times 0.9 \times 0.8 = 720$|

**Observed table:**

||Term occurs|Term does not occur|
|---|---|---|
|User watched item|$O_1 = 60$|$O_2 = 40$|
|User did not watch|$O_3 = 140$|$O_4 = 760$|

$$ \chi^2 = \sum_{i=1}^{p} \frac{(O_i - E_i)^2}{E_i} = \frac{(60-20)^2}{20} + \frac{(40-80)^2}{80} + \frac{(140-180)^2}{180} + \frac{(760-720)^2}{720} $$

A high $\chi^2$ means the feature is **predictive** of the user's behaviour → keep it; assign it a higher weight.

#### For binary / small discrete ratings — Gini & Entropy

$$ \text{Gini}(w) = 1 - \sum_{i=1}^{t} p_i(w)^2 $$

$$ \text{Entropy}(w) = -\sum_{i=1}^{t} p_i(w)\log\bigl(p_i(w)\bigr) $$

where $p_i(w)$ is the probability of rating class $i$ among items containing word $w$.

#### For higher-cardinality discrete ratings — Deviation

$$ \text{Dev}(w) = \frac{\bigl|\mu^+(w) - \mu^-(w)\bigr|}{\sigma} $$

- $\mu^+(w)$: mean rating of items **containing** word $w$
- $\mu^-(w)$: mean rating of items **not containing** word $w$
- $\sigma$: pooled standard deviation

High deviation → the word strongly separates liked from disliked items.

### 4.4 Modelling

|Rating type|ML task|Models (via scikit-learn)|
|---|---|---|
|Binary (liked/not liked)|**Classification**|Logistic Regression, SVM, Random Forest, Naïve Bayes…|
|Discrete / continuous|**Regression**|Linear Regression, Ridge, SVR, Gradient Boosting…|

The user profile is the **trained model itself** (its weights, decision boundary, or regression coefficients).

### 4.5 Three Representations of a User Profile

#### Example 1 — Classifier Boundary

The profile is a **decision boundary** in item feature space. Items on the positive side are recommended.

```
Feature 2
    ▲         Pinocchio ●
    │        ╱
    │       ╱ ← Bob's profile (boundary)
    │      ╱    ● Black Mirror
    │     ╱     ● Terminator
    └────────────────► Feature 1
         Dislike │ Like
```

#### Example 2 — User Profile Vector

The profile is a **vector** — a weighted average of the item vectors the user liked. Recommendation = find items closest (by cosine similarity) to this vector.

$$ \vec{p}_{\text{user}} = \frac{\sum_{i \in \text{rated}} r_i \cdot \vec{v}_i}{\sum_{i \in \text{rated}} r_i} $$

#### Example 3 — Regression Coefficients

The profile is a **regression line** from content features → predicted rating.

$$\hat{r}_{ui} = \vec{\beta}_u \cdot \vec{v}_i + \beta_0$$

where $\vec{\beta}_u$ are the learned coefficients (the user's "taste weights" per feature).

### 4.6 Building an Explainable Profile (Weighted Average Method)

Given item feature matrix and user ratings:

|Item|`is_drama`|`is_sci`|`year` (norm)|`topic_0`|Rating|
|---|---|---|---|---|---|
|Black Mirror|1|1|0.50|1|⭐⭐⭐⭐⭐ (5)|
|Ex Machina|0|1|0.80|0|⭐⭐⭐⭐ (4)|
|Toy Story|0|0|0.30|0|unrated|
|Shrek|0|0|0.20|1|⭐ (1)|

User profile = **rating-weighted average** (unrated items excluded):

$$ p[\texttt{year}] = \frac{5 \times 0.50 + 4 \times 0.80 + 1 \times 0.20}{5 + 4 + 1} = \frac{2.5 + 3.2 + 0.2}{10} = \frac{5.9}{10} = 0.59 $$

Full profile:

|`is_drama`|`is_sci`|`year`|`topic_0`|
|---|---|---|---|
|0.50|0.90|**0.59**|0.60|

This profile is **explainable**: high `is_sci` → _"Because you like Science Fiction"_.

---

## 5. The Filtering Component

The filtering component **scores new items** against the user profile and returns a ranked list.

- **Classifier approach:** items on the positive side of the decision boundary are recommended; rank by distance to the boundary (e.g., probability score).
- **Vector approach:** compute [[cosine similarity]] between user profile vector and each unseen item vector:

$$ \text{sim}(\vec{p}_u, \vec{v}_i) = \frac{\vec{p}_u \cdot \vec{v}_i}{|\vec{p}_u| , |\vec{v}_i|} $$

Recommend the top-$N$ items by similarity.

- **Regression approach:** predict $\hat{r}_{ui}$ for all unseen items and recommend the top-$N$ by predicted rating.

---

## 6. Useful Datasets for Movie Content-Based RS

|Paper|Content provided|
|---|---|
|_The Tag Genome: Encoding Community Knowledge to Support Novel Interaction_ (Vig, Sen & Riedl)|**Tag scores** — continuous relevance scores of tags per movie|
|_Using Visual Features and Latent Factors for Movie Recommendation_ (Deldjoo et al.)|**Visual features** — lighting, colours, motion extracted from movie frames|

> 🗂️ These datasets are compatible with the MovieLens dataset and can enrich your movie recommender project.

---

## 7. Pros and Cons

### ✅ Advantages

|Advantage|Explanation|
|---|---|
|**Handles new items**|No ratings needed; only item attributes|
|**Handles unpopular items**|No co-occurrence problem|
|**Explainability**|Profile features have semantic meaning ("Because you like Sci-Fi")|
|**User independence**|Each user's model is independent — no privacy leakage between users|
|**Off-the-shelf models**|Any scikit-learn classifier/regressor works|

### ❌ Disadvantages

|Disadvantage|Explanation|
|---|---|
|**New user problem**|Cold start: without any ratings, no profile can be built|
|**Over-specialisation**|Recommends only items similar to what the user already knows — no serendipity|
|**Domain knowledge required**|You need to know which features to extract and how|
|**NLP limits**|Text mining is imperfect; irony, synonymy, polysemy are hard to handle|

---

## 8. Key Formulas Summary

|Formula|Purpose|
|---|---|
|$\text{TF-IDF}(w,d,D) = \frac{f_{w,d}}{\sum_{w'}f_{w',d}} \cdot \log\frac{|D|
|$\chi^2 = \sum_i \frac{(O_i - E_i)^2}{E_i}$|Test if a feature predicts user behaviour (binary ratings)|
|$\text{Gini}(w) = 1 - \sum_i p_i(w)^2$|Feature importance for discrete ratings|
|$\text{Entropy}(w) = -\sum_i p_i(w)\log p_i(w)$|Feature importance for discrete ratings|
|$\text{Dev}(w) = \frac{|\mu^+(w) - \mu^-(w)|
|$\vec{p}_u = \frac{\sum_i r_i \vec{v}_i}{\sum_i r_i}$|Rating-weighted user profile vector|
|$\text{sim}(\vec{p}_u, \vec{v}_i) = \frac{\vec{p}_u \cdot \vec{v}_i}{\|\vec{p}_u\|\vec{v}_i\|}$|Cosine similarity for ranking|
|$\hat{r}_{ui} = \vec{\beta}_u \cdot \vec{v}_i + \beta_0$|Regression-based rating prediction|

---

## 9. Related Notes

- [[Chapter 3 — Collaborative Filtering]]
- [[Cosine Similarity]]
- [[TF-IDF]]
- [[Latent Dirichlet Allocation (LDA)]]
- [[Cold Start Problem]]
- [[Feature Engineering]]
- [[Recommender System Models — Overview]]

---

## 📚 References

- Ricci, F. et al. _Recommender Systems Handbook_. Springer, 2011.
- Falk, K. _Practical Recommender Systems_. Simon and Schuster, 2019.
- Vig, J., Sen, S., & Riedl, J. _The Tag Genome: Encoding Community Knowledge to Support Novel Interaction_. ACM TIIS, 2012.
- Deldjoo, Y., Elahi, M., & Cremonesi, P. _Using Visual Features and Latent Factors for Movie Recommendation_. CBRecSys, 2016.