# Edit Distance (Levenshtein) for Gesture Recognition

---
tags:
  - gesture-recognition
  - baseline
  - edit-distance
  - sequence-quantization
  - dynamic-programming
  - sequence-classification
---

> **Role in study:** Baseline method  
> **Key papers:** [[Papers/Levenshtein1966|Levenshtein (1966)]], [[Papers/Wagner1974|Wagner & Fischer (1974)]], [[Papers/Nyirarugira2014|Nyirarugira (2014)]], [[Papers/GarciaDiez2011_SumOverPaths|Garcia-Diez et al. (2011)]], [[Papers/Theodoridis2009|Theodoridis & Koutroumbas (2009)]], [[Papers/Gavrila1999|Gavrila (1999)]]  
> **Concepts:** [[Concepts/Dynamic_Programming|Dynamic Programming]], [[Concepts/Vector_Quantization|Vector Quantization]], [[Concepts/Clustering_EM|Clustering & k-Means]], [[Concepts/kNN_Classifier|k-NN Classifier]], [[Concepts/Cross_Validation|Cross-Validation Protocol]], [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]], [[Concepts/Longest_Common_Subsequence|LCS (related)]]  
> **Workflows:** [[Workflows/Data_Preprocessing|Data Preprocessing]], [[Workflows/Model_Evaluation|Model Evaluation]]  
> **Related methods:** [[04_DTW_kNN|DTW]], [[06_Three_Cent_Recognizer|$3 Recognizer]], [[07_BiLSTM|BiLSTM]]

---

## 1. Conceptual Overview

Edit distance (also known as **Levenshtein distance**) measures the dissimilarity between two symbolic sequences as the minimum number of elementary edit operations — **insertions**, **deletions**, and **substitutions** — needed to transform one sequence into the other.

Applied to gesture recognition, the continuous time-series of joint coordinates must first be **discretized** (quantized) into a sequence of symbols from a finite alphabet. Once gestures are represented as strings, standard edit distance algorithms (e.g., Wagner-Fischer dynamic programming) can be used as a similarity measure for classification.

The method was popularized in the gesture domain by Nyirarugira & Choi (2014), who demonstrated real-time recognition at 93.9% accuracy by addressing temporal variability through the edit distance's inherent alignment flexibility — without requiring explicit time-warping.

---

## 2. Pipeline Architecture

The full pipeline consists of three stages:

```
Continuous gesture sequence (T, D)
         │
         ▼
   ┌─────────────┐
   │ Quantization │  Map each frame to a discrete symbol
   │  (Codebook)  │  from a finite alphabet Σ
   └──────┬──────┘
          │
          ▼
   String S ∈ Σ*          (e.g., "AABCDDCBA")
          │
          ▼
   ┌──────────────────┐
   │  Edit Distance   │  Wagner-Fischer DP
   │  against all     │  between input string
   │  class templates │  and all stored templates
   └──────┬───────────┘
          │
          ▼
   1-NN classification: label of nearest template
```

---

## 3. Stage 1: Sequence Quantization

### 3.1 Why Quantize?

Edit distance is defined on symbolic strings. The continuous feature vectors (e.g., 3D joint positions) must be mapped to a discrete codebook of $K$ symbols. This is equivalent to **vector quantization (VQ)**.

### 3.2 k-Means Codebook Construction

Given all training frames $\{x_t\}_{t=1}^{N_\text{total}} \in \mathbb{R}^D$, cluster them into $K$ clusters using k-means. The cluster centers $\{c_1, \ldots, c_K\}$ form the codebook. Each frame $x_t$ is then encoded as:

$$\text{symbol}(x_t) = \underset{k \in \{1,\ldots,K\}}{\arg\min}\ \|x_t - c_k\|_2$$

The gesture becomes a string $S = s_1 s_2 \cdots s_T$ with $s_i \in \{1, \ldots, K\}$.

### 3.3 Resampling Before Quantization

Quantization is typically applied **after** resampling the sequence to a fixed length, ensuring that the codebook is not dominated by frames from long gestures.

---

## 4. Stage 2: Edit Distance Computation

### 4.1 Levenshtein Distance (Uniform Costs)

Let $S = s_1 \cdots s_n$ and $T = t_1 \cdots t_m$ be two strings. The Levenshtein distance $\text{lev}(S, T)$ is defined recursively:

$$\text{lev}(i, j) = \begin{cases} i & \text{if } j = 0 \\ j & \text{if } i = 0 \\ \text{lev}(i-1, j-1) & \text{if } s_i = t_j \\ 1 + \min\begin{cases} \text{lev}(i-1, j) & (\text{deletion}) \\ \text{lev}(i, j-1) & (\text{insertion}) \\ \text{lev}(i-1, j-1) & (\text{substitution}) \end{cases} & \text{if } s_i \neq t_j \end{cases}$$

**Complexity:** $\mathcal{O}(nm)$ time and $\mathcal{O}(\min(n,m))$ space (with the two-row trick).

### 4.2 Weighted Edit Distance

A key extension for gesture recognition is to use **distance-proportional substitution costs** rather than uniform cost 1. When substituting symbol $a$ for symbol $b$, the cost reflects how far the corresponding codebook centers are:

$$\text{cost}_\text{sub}(a, b) = \frac{\|c_a - c_b\|_2}{\max_{p,q} \|c_p - c_q\|_2}$$

This prevents treating all mismatches equally — replacing a symbol with a "nearby" codeword is cheaper than replacing it with a completely different one.

### 4.3 Modified Levenshtein for Temporal Variability (Nyirarugira & Choi, 2014)

Nyirarugira & Choi observed that standard uniform edit distance is sensitive to temporal variability in gesture speed. Their modification represents each gesture class by a **single exemplar** and allows for implicit time stretching by allowing adjacent repeated symbols to match without cost:

$$\text{cost}_\text{rep}(s_i, t_j) = 0 \quad \text{if } s_i = t_j$$
$$\text{cost}_\text{rep}(s_i, t_j) = 1 \quad \text{otherwise}$$

Combined with variable insertion/deletion costs, this allowed their system to handle gestures of variable speed without requiring explicit resampling.

---

## 5. Classification

As with DTW, classification uses **1-Nearest Neighbor**: the input gesture string is compared against all stored template strings, and the class with the smallest edit distance is returned.

For multi-template settings (multiple exemplars per class), the class score is computed as the minimum edit distance over all class templates:

$$\hat{y} = \underset{c}{\arg\min}\ \min_{T \in \mathcal{T}_c} \text{lev}(S_\text{input}, T)$$

---

## 6. Model: Python Implementation

### 6.1 Dependencies

```bash
pip install numpy scikit-learn editdistance
```

> **Repository references:**
> - `editdistance` (C++ backed, fast): https://github.com/roy-ht/editdistance
> - `rapidfuzz` (also fast, weighted Levenshtein): https://github.com/maxbachmann/RapidFuzz

### 6.2 Full Implementation

```python
import numpy as np
from sklearn.cluster import KMeans, MiniBatchKMeans
from sklearn.metrics import accuracy_score
import editdistance  # Fast C++ Levenshtein implementation


# ──────────────────────────────────────────────────────
# 1. Preprocessing & Resampling
# ──────────────────────────────────────────────────────

def resample_sequence(seq: np.ndarray, target_len: int) -> np.ndarray:
    """Resample (T, D) → (target_len, D) via linear interpolation."""
    T, D = seq.shape
    old_t = np.linspace(0, 1, T)
    new_t = np.linspace(0, 1, target_len)
    return np.stack([np.interp(new_t, old_t, seq[:, d]) for d in range(D)], axis=1)


# ──────────────────────────────────────────────────────
# 2. Codebook Construction (Vector Quantization)
# ──────────────────────────────────────────────────────

class GestureCodebook:
    """
    Builds a vector quantization codebook from training frames.
    Maps continuous feature vectors to discrete symbol indices.
    """
    
    def __init__(self, n_symbols: int = 16, method: str = "kmeans", random_state: int = 42):
        """
        Args:
            n_symbols: Codebook size K (alphabet cardinality)
            method: "kmeans" or "minibatch_kmeans"
            random_state: RNG seed for reproducibility
        """
        self.n_symbols  = n_symbols
        self.method     = method
        self.random_state = random_state
        self.codebook_  = None  # Cluster centers (K, D)

    def fit(self, sequences: list[np.ndarray]) -> "GestureCodebook":
        """
        Fit codebook on all frames from training sequences.
        Args:
            sequences: List of (T_i, D) arrays (already resampled)
        """
        all_frames = np.concatenate(sequences, axis=0)  # (N_total_frames, D)
        
        if self.method == "minibatch_kmeans":
            km = MiniBatchKMeans(n_clusters=self.n_symbols,
                                 random_state=self.random_state,
                                 n_init=10)
        else:
            km = KMeans(n_clusters=self.n_symbols,
                        random_state=self.random_state,
                        n_init=10)
        
        km.fit(all_frames)
        self.codebook_ = km.cluster_centers_  # (K, D)
        return self

    def encode(self, seq: np.ndarray) -> list[int]:
        """
        Map a (T, D) sequence to a list of integer symbol indices.
        Uses nearest-centroid assignment.
        """
        # Pairwise distances: (T, K)
        dists = np.linalg.norm(seq[:, None, :] - self.codebook_[None, :, :], axis=2)
        return dists.argmin(axis=1).tolist()

    def substitution_cost_matrix(self) -> np.ndarray:
        """
        Build a (K, K) matrix of normalized inter-centroid distances.
        Used for weighted edit distance.
        """
        K = self.n_symbols
        C = self.codebook_
        costs = np.zeros((K, K))
        for i in range(K):
            for j in range(K):
                costs[i, j] = np.linalg.norm(C[i] - C[j])
        max_cost = costs.max()
        return costs / (max_cost + 1e-8)


# ──────────────────────────────────────────────────────
# 3. Edit Distance Computation
# ──────────────────────────────────────────────────────

def levenshtein_uniform(s: list[int], t: list[int]) -> int:
    """
    Standard Levenshtein distance with uniform costs (ins=del=sub=1).
    Uses fast C++ backend via editdistance library.
    """
    return editdistance.eval(s, t)


def levenshtein_weighted(s: list[int], t: list[int],
                          sub_costs: np.ndarray,
                          ins_cost: float = 1.0,
                          del_cost: float = 1.0) -> float:
    """
    Weighted Levenshtein distance with distance-proportional substitution costs.
    Pure Python DP — slower, use only for ablation studies.
    
    Args:
        s, t: Symbol sequences (lists of int indices)
        sub_costs: (K, K) substitution cost matrix from codebook
        ins_cost: Cost of inserting a symbol
        del_cost: Cost of deleting a symbol
    """
    n, m = len(s), len(t)
    # DP matrix
    D = np.zeros((n + 1, m + 1))
    D[:, 0] = np.arange(n + 1) * del_cost
    D[0, :] = np.arange(m + 1) * ins_cost
    
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            cost_sub = 0.0 if s[i-1] == t[j-1] else sub_costs[s[i-1], t[j-1]]
            D[i, j] = min(
                D[i-1, j]   + del_cost,   # deletion
                D[i, j-1]   + ins_cost,   # insertion
                D[i-1, j-1] + cost_sub,   # substitution
            )
    return float(D[n, m])


# ──────────────────────────────────────────────────────
# 4. Classifier
# ──────────────────────────────────────────────────────

class EditDistanceClassifier:
    """
    1-Nearest Neighbor classifier using Edit Distance on quantized gesture strings.
    """
    
    def __init__(self,
                 n_symbols: int = 16,
                 target_len: int = 64,
                 weighted: bool = False,
                 ins_cost: float = 1.0,
                 del_cost: float = 1.0,
                 codebook_method: str = "kmeans"):
        self.n_symbols       = n_symbols
        self.target_len      = target_len
        self.weighted        = weighted
        self.ins_cost        = ins_cost
        self.del_cost        = del_cost
        self.codebook_method = codebook_method
        
        self.codebook_       = None
        self.train_strings_  = []
        self.train_labels_   = []
        self.sub_costs_      = None

    def fit(self, sequences: list[np.ndarray], labels: np.ndarray):
        """
        Build codebook from training sequences and encode all training gestures.
        Args:
            sequences: List of variable-length (T_i, D) arrays
            labels: Class labels of shape (N,)
        """
        # 1. Resample
        resampled = [resample_sequence(s, self.target_len) for s in sequences]
        
        # 2. Build codebook
        self.codebook_ = GestureCodebook(
            n_symbols=self.n_symbols,
            method=self.codebook_method
        ).fit(resampled)
        
        # 3. Encode all training sequences
        self.train_strings_ = [self.codebook_.encode(s) for s in resampled]
        self.train_labels_  = list(labels)
        
        # 4. Precompute substitution cost matrix for weighted mode
        if self.weighted:
            self.sub_costs_ = self.codebook_.substitution_cost_matrix()
        
        return self

    def predict(self, sequences: list[np.ndarray]) -> np.ndarray:
        """
        Classify each sequence by 1-NN edit distance against training strings.
        """
        predictions = []
        for seq in sequences:
            seq_r   = resample_sequence(seq, self.target_len)
            sym_seq = self.codebook_.encode(seq_r)
            
            # Compute distance to each training sample
            min_dist = float("inf")
            best_lbl = -1
            
            for train_str, lbl in zip(self.train_strings_, self.train_labels_):
                if self.weighted:
                    d = levenshtein_weighted(sym_seq, train_str,
                                             self.sub_costs_,
                                             self.ins_cost,
                                             self.del_cost)
                else:
                    d = levenshtein_uniform(sym_seq, train_str)
                
                if d < min_dist:
                    min_dist = d
                    best_lbl = lbl
            
            predictions.append(best_lbl)
        return np.array(predictions)

    def score(self, sequences: list[np.ndarray], labels: np.ndarray) -> float:
        preds = self.predict(sequences)
        return accuracy_score(labels, preds)


# ──────────────────────────────────────────────────────
# 5. Usage Example
# ──────────────────────────────────────────────────────

if __name__ == "__main__":
    from sklearn.model_selection import StratifiedKFold
    
    np.random.seed(42)
    N_CLASSES, N_PER_CLASS = 14, 5
    N = N_CLASSES * N_PER_CLASS
    
    raw_seqs = [np.random.randn(np.random.randint(40, 80), 63) for _ in range(N)]
    labels   = np.repeat(np.arange(N_CLASSES), N_PER_CLASS)
    
    skf  = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    accs = []
    
    for fold, (train_idx, test_idx) in enumerate(skf.split(raw_seqs, labels)):
        train_seqs = [raw_seqs[i] for i in train_idx]
        test_seqs  = [raw_seqs[i] for i in test_idx]
        
        clf = EditDistanceClassifier(
            n_symbols=16,
            target_len=64,
            weighted=True,
            ins_cost=1.0,
            del_cost=1.0,
        )
        clf.fit(train_seqs, labels[train_idx])
        acc = clf.score(test_seqs, labels[test_idx])
        accs.append(acc)
        print(f"Fold {fold+1}: {acc:.4f}")
    
    print(f"\nMean accuracy: {np.mean(accs):.4f} ± {np.std(accs):.4f}")
```

---

## 7. Parameters & Ablation Guide

| Parameter | Candidates | Default | Effect |
|---|---|---|---|
| `n_symbols` (Codebook size K) | 8, **16**, 32, 64 | 16 | Larger K → finer discretization, richer alphabet, slower |
| `target_len` (Resampling) | 32, **64**, 128 | 64 | Affects string length and thus edit distance scale |
| `codebook_method` | **`kmeans`**, `minibatch_kmeans` | `kmeans` | Quality vs speed trade-off for codebook construction |
| `weighted` | **False**, True | False | Uniform vs distance-proportional substitution costs |
| `ins_cost` | 0.5, **1.0**, 2.0 | 1.0 | Cost of inserting a symbol (affects alignment preference) |
| `del_cost` | 0.5, **1.0**, 2.0 | 1.0 | Cost of deleting a symbol (affects alignment preference) |

### Parameter Explanations

**`n_symbols` (Alphabet size / Codebook size $K$)**
The most influential parameter. A small $K$ (e.g., 8) produces coarse representations where many distinct frames share a symbol — high tolerance to noise but reduced discriminability. A large $K$ (e.g., 64) gives richer representations but makes the edit distance larger on average and is more sensitive to noise.

**`target_len`**
Before encoding, sequences are resampled to a fixed length. This controls the string length fed to edit distance. Longer strings carry more temporal detail but make the DP more expensive and increase the absolute range of edit distances.

**`weighted`**
Uniform Levenshtein treats all mismatches equally (cost 1). Weighted edit distance uses the normalized Euclidean distance between the two substituted codebook centers as the substitution cost. This softens the penalty for replacing a symbol with a perceptually close one, which often helps when codebook boundaries are not clearly separated.

**`ins_cost` / `del_cost`**
Asymmetric insertion/deletion costs can encode domain priors. For instance, if gesture execution is typically slower than the template (more frames), increasing `ins_cost` (penalize extra insertions) may help. Equal costs (1.0, 1.0) are the standard Levenshtein baseline.

---

## 8. Comparison with DTW

| Aspect | DTW | Edit Distance |
|---|---|---|
| Input type | Continuous features | Discrete symbols |
| Alignment | Elastic time warping | Symbolic edit ops |
| Complexity | $\mathcal{O}(nm)$ frames | $\mathcal{O}(nm)$ symbols, but shorter |
| Learned component | None | Codebook via k-means |
| Sensitivity to noise | Frame-level noise | Quantization noise |
| Speed | Slower (float ops) | Faster (integer comparison) |

---

## 9. Limitations

- **Quantization loss:** Information is lost when projecting continuous frames to discrete symbols. A poor codebook (too few clusters, bad initialization) hurts downstream recognition.
- **Codebook leakage:** If the codebook is fitted on the full dataset (including test), results are optimistic. Always fit codebook on training folds only.
- **String length sensitivity:** Edit distance is not normalized by string length by default, making distances incomparable across sequences of very different lengths unless normalized: $\text{lev\_norm}(S,T) = \text{lev}(S,T) / \max(|S|, |T|)$.

---

## 10. References

> See `gesture_recognition.bib` for importable BibTeX entries.

- Levenshtein, V. I. (1966). Binary codes capable of correcting deletions, insertions, and reversals. *Soviet Physics Doklady*, 10(8), 707–710. → [[Papers/Levenshtein1966|Levenshtein1966]]
- Wagner, R. A., & Fischer, M. J. (1974). The string-to-string correction problem. *Journal of the ACM*, 21(1), 168–173. → [[Papers/Wagner1974|Wagner1974]]
- Nyirarugira, C., & Choi, T. (2014). Modified Levenshtein distance for real-time gesture recognition. *IEEE ICASSP 2014*. → [[Papers/Nyirarugira2014|Nyirarugira2014]]
- Gavrila, D. M. (1999). The visual analysis of human movement: A survey. *Computer Vision and Image Understanding*, 73(1), 82–98. → [[Papers/Gavrila1999|Gavrila1999]]
- Theodoridis, S., & Koutroumbas, K. (2009). *Pattern Recognition, 4th ed.* Academic Press. → [[Papers/Theodoridis2009|Theodoridis2009]] *(required citation per guidelines)*
