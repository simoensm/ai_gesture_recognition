# Dynamic Time Warping (DTW)

---
tags:
  - gesture-recognition
  - baseline
  - dtw
  - time-series
  - dynamic-programming
  - sequence-classification
---

> **Role in study:** Baseline method  
> **Key papers:** [[Papers/Sakoe1978|Sakoe & Chiba (1978)]], [[Papers/Berndt1994|Berndt & Clifford (1994)]], [[Papers/Keogh2001_DDTW|Keogh & Pazzani (2001)]], [[Papers/Salvador2007|Salvador & Chan (2007)]], [[Papers/Rabiner1993_SpeechRecognition|Rabiner & Juang (1993)]], [[Papers/Itakura1975|Itakura (1975)]]  
> **Concepts:** [[Concepts/Dynamic_Programming|Dynamic Programming]], [[Concepts/kNN_Classifier|k-NN Classifier]], [[Concepts/Cross_Validation|Cross-Validation Protocol]], [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]]  
> **Workflows:** [[Workflows/Data_Preprocessing|Data Preprocessing]], [[Workflows/Model_Evaluation|Model Evaluation]], [[Workflows/Hyperparameter_Tuning|Hyperparameter Tuning]]  
> **Related methods:** [[05_Edit_Distance_kNN|Edit Distance]], [[06_Three_Cent_Recognizer|$3 Recognizer]], [[07_BiLSTM|BiLSTM]]

---

## 1. Conceptual Overview

Dynamic Time Warping is a similarity measure between two time series that allows elastic alignment along the time axis. Unlike Euclidean distance, which compares sequences point-by-point at identical time steps, DTW finds the optimal non-linear mapping between two sequences, tolerating temporal distortions such as variations in speed, duration, or rhythm.

Originally developed by Sakoe & Chiba (1978) for spoken word recognition in speech processing, DTW became a cornerstone baseline in gesture and activity recognition due to its simplicity and robustness to temporal variability — a key challenge in gesture recognition where the same gesture may be performed at varying speeds by different users.

---

## 2. Formal Definition

Let $\mathbf{x} = (x_1, x_2, \ldots, x_n)$ and $\mathbf{y} = (y_1, y_2, \ldots, y_m)$ be two time series of lengths $n$ and $m$, where each element $x_i, y_j \in \mathbb{R}^d$ lives in a $d$-dimensional feature space.

### 2.1 Cost Matrix

Construct an $(n \times m)$ local cost matrix $C$ where:

$$C(i, j) = \delta(x_i, y_j)$$

Here $\delta$ is a local distance function (typically Euclidean distance). The entry $C(i,j)$ measures the cost of aligning frame $i$ of $\mathbf{x}$ with frame $j$ of $\mathbf{y}$.

### 2.2 Warping Path

A **warping path** $W = (w_1, w_2, \ldots, w_K)$ is a sequence of index pairs $w_k = (i_k, j_k)$ satisfying:

1. **Boundary conditions:** $w_1 = (1,1)$ and $w_K = (n,m)$
2. **Monotonicity:** $i_{k+1} \geq i_k$ and $j_{k+1} \geq j_k$
3. **Step constraint:** $w_{k+1} - w_k \in \{(1,0), (0,1), (1,1)\}$

The total cost of a warping path is:

$$\text{cost}(W) = \sum_{k=1}^{K} C(i_k, j_k)$$

### 2.3 DTW Distance

The DTW distance is the cost of the **optimal** (minimum-cost) warping path:

$$\text{DTW}(\mathbf{x}, \mathbf{y}) = \min_{W} \sum_{k=1}^{K} C(i_k, j_k)$$

This is computed via dynamic programming using the recurrence:

$$D(i, j) = C(i, j) + \min\{D(i-1, j),\ D(i, j-1),\ D(i-1, j-1)\}$$

with $D(0,0) = 0$ and $D(i,0) = D(0,j) = +\infty$ for $i,j > 0$. The DTW distance is then $D(n,m)$.

**Time complexity:** $\mathcal{O}(nm)$ — quadratic in sequence length, which is the main computational bottleneck.

---

## 3. Key Algorithmic Contributions from the Literature

### 3.1 Sakoe-Chiba Band (Sakoe & Chiba, 1978)

To prevent pathological alignments (e.g., a single frame matched to an entire sequence) and to reduce computation, Sakoe & Chiba introduced a **warping window** constraint parameterized by a radius $r$:

$$|i - j| \leq r$$

Only cells within this diagonal band are computed, reducing complexity to $\mathcal{O}(rn)$.

They also studied the **slope constraint**, limiting how steeply the warping path can ascend or descend, and concluded that the **symmetric** DTW formulation with slope constraints outperforms the asymmetric form.

### 3.2 Itakura Parallelogram ([[Papers/Itakura1975|Itakura, 1975]])

An alternative path constraint that limits the slope to a maximum value $s$, forming a parallelogram-shaped accessible region in the cost matrix. Less commonly used than Sakoe-Chiba in modern gesture recognition but historically important.

### 3.3 Derivative DTW — DDTW (Keogh & Pazzani, 2001)

Keogh & Pazzani observe that standard DTW can produce counter-intuitive alignments because it compares absolute amplitude values. They propose **Derivative DTW**, which replaces each point $x_i$ with an estimate of its first derivative:

$$\hat{x}_i = \frac{(x_i - x_{i-1}) + \frac{(x_{i+1} - x_{i-1})}{2}}{2}$$

The local cost becomes $\delta(\hat{x}_i, \hat{y}_j)$. This makes DTW sensitive to **shape** rather than offset, which is more discriminative for gesture trajectories.

### 3.4 FastDTW (Salvador & Chan, 2007)

A multilevel approximation of DTW that achieves $\mathcal{O}(n)$ time and space via:
1. **Coarsening:** recursively downsample the sequences
2. **Projection:** use coarse-level path to constrain fine-level search window
3. **Refinement:** iteratively refine to full resolution

FastDTW trades a small accuracy loss for dramatically faster computation — critical when comparing against many templates.

### 3.5 Weighted DTW for Gesture Recognition

For multi-joint skeleton data, Reyes et al. (2011) propose weighting each joint by a discriminant ratio optimized during training. Joints that better separate gesture classes receive higher weights in the local cost:

$$C(i,j) = \sum_{k=1}^{J} w_k \cdot \delta(x_i^{(k)}, y_j^{(k)})$$

where $J$ is the number of joints and $w_k$ are learned weights.

---

## 4. DTW in Gesture Recognition: Classification Pipeline

```
Input gesture sequence
        │
        ▼
Preprocessing
  - Resampling to fixed length L
  - Normalization (zero-mean, unit-variance per axis)
  - Optional: velocity/derivative features
        │
        ▼
For each template in library:
  Compute DTW(input, template)
        │
        ▼
1-NN classification:
  Assign label of nearest template (min DTW distance)
```

DTW is used as a **nearest-neighbor** (1-NN) classifier: the input gesture is assigned the label of the training sample with the smallest DTW distance. Despite its simplicity, 1-NN DTW is a remarkably strong baseline and difficult to beat on many benchmark datasets (Ding et al., 2008).

---

## 5. Model: Python Implementation

### 5.1 Dependencies

```bash
pip install tslearn numpy scikit-learn
# For FastDTW:
pip install fastdtw
```

> **Repository references:**
> - `tslearn`: https://github.com/tslearn-team/tslearn — provides DTW with Sakoe-Chiba/Itakura constraints, kNN classifier, barycenter averaging
> - `dtaidistance`: https://github.com/wannesm/dtaidistance — fast C-accelerated DTW, supports multivariate
> - `fastdtw`: https://github.com/slaypni/fastdtw — approximate linear-time DTW

### 5.2 Core Implementation

```python
import numpy as np
from tslearn.metrics import dtw, dtw_path
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import accuracy_score, classification_report


# ──────────────────────────────────────────────────────
# 1. Preprocessing
# ──────────────────────────────────────────────────────

def resample_sequence(seq: np.ndarray, target_len: int) -> np.ndarray:
    """
    Resample a (T, D) sequence to (target_len, D) using linear interpolation.
    
    Args:
        seq: Input sequence of shape (T, D)
        target_len: Desired number of time steps
    Returns:
        Resampled sequence of shape (target_len, D)
    """
    T, D = seq.shape
    old_t = np.linspace(0, 1, T)
    new_t = np.linspace(0, 1, target_len)
    return np.stack([np.interp(new_t, old_t, seq[:, d]) for d in range(D)], axis=1)


def normalize_sequence(seq: np.ndarray) -> np.ndarray:
    """Zero-mean, unit-variance normalization per feature dimension."""
    mean = seq.mean(axis=0, keepdims=True)
    std  = seq.std(axis=0, keepdims=True) + 1e-8
    return (seq - mean) / std


def compute_derivatives(seq: np.ndarray) -> np.ndarray:
    """
    Compute estimated first derivative (Keogh & Pazzani, 2001 — DDTW).
    Output has same shape as input (endpoints use one-sided differences).
    """
    D_seq = np.zeros_like(seq)
    # Central differences for interior points
    D_seq[1:-1] = ((seq[1:-1] - seq[:-2]) + (seq[2:] - seq[:-2]) / 2) / 2
    # One-sided differences at boundaries
    D_seq[0]    = seq[1]  - seq[0]
    D_seq[-1]   = seq[-1] - seq[-2]
    return D_seq


def preprocess(sequences: list[np.ndarray],
               target_len: int = 64,
               use_derivatives: bool = False,
               normalize: bool = True) -> np.ndarray:
    """
    Full preprocessing pipeline.
    
    Args:
        sequences: List of variable-length (T_i, D) arrays
        target_len: Fixed resampling length
        use_derivatives: Append derivative features
        normalize: Apply per-sequence z-score normalization
    Returns:
        Array of shape (N, target_len, D) or (N, target_len, 2D) with derivatives
    """
    processed = []
    for seq in sequences:
        seq = resample_sequence(seq, target_len)
        if normalize:
            seq = normalize_sequence(seq)
        if use_derivatives:
            deriv = compute_derivatives(seq)
            seq = np.concatenate([seq, deriv], axis=1)
        processed.append(seq)
    return np.array(processed)


# ──────────────────────────────────────────────────────
# 2. DTW Classifier (1-NN)
# ──────────────────────────────────────────────────────

class DTWClassifier:
    """
    1-Nearest Neighbor classifier using DTW distance.
    Wraps tslearn's KNeighborsTimeSeriesClassifier.
    """
    
    def __init__(self,
                 window: float | None = None,
                 constraint: str = "sakoe_chiba",
                 metric: str = "dtw",
                 n_neighbors: int = 1):
        """
        Args:
            window: Sakoe-Chiba band radius as a fraction of sequence length.
                    None = unconstrained. E.g. 0.1 = 10% of sequence length.
            constraint: "sakoe_chiba" or "itakura"
            metric: "dtw" (standard) or "softdtw"
            n_neighbors: Number of neighbors (1 for pure 1-NN)
        """
        self.window = window
        self.constraint = constraint
        self.n_neighbors = n_neighbors
        
        # Build metric_params
        if window is not None:
            self.metric_params = {
                "global_constraint": constraint,
                "sakoe_chiba_radius": None,  # will be set after seeing data
            }
        else:
            self.metric_params = {}
        
        self.clf = None
        self.train_len = None

    def fit(self, X: np.ndarray, y: np.ndarray):
        """
        Args:
            X: Training data of shape (N, T, D)
            y: Labels of shape (N,)
        """
        self.train_len = X.shape[1]
        
        # Convert fractional window to integer radius
        if self.window is not None:
            radius = int(self.window * self.train_len)
            self.metric_params = {
                "global_constraint": self.constraint,
                "sakoe_chiba_radius": radius,
            }
        
        self.clf = KNeighborsTimeSeriesClassifier(
            n_neighbors=self.n_neighbors,
            metric="dtw",
            metric_params=self.metric_params if self.metric_params else None,
        )
        self.clf.fit(X, y)
        return self

    def predict(self, X: np.ndarray) -> np.ndarray:
        return self.clf.predict(X)

    def score(self, X: np.ndarray, y: np.ndarray) -> float:
        return self.clf.score(X, y)


# ──────────────────────────────────────────────────────
# 3. Usage Example
# ──────────────────────────────────────────────────────

if __name__ == "__main__":
    from sklearn.model_selection import StratifiedKFold
    
    # -- Simulate data --
    # Replace with your actual data loading
    np.random.seed(42)
    N_CLASSES = 14
    N_SAMPLES = 5   # samples per class
    N         = N_CLASSES * N_SAMPLES
    
    # Variable-length sequences (T between 40 and 80 frames, 63 features = 21 joints × 3)
    raw_seqs  = [np.random.randn(np.random.randint(40, 80), 63) for _ in range(N)]
    labels    = np.repeat(np.arange(N_CLASSES), N_SAMPLES)
    
    # -- Preprocessing --
    TARGET_LEN   = 64
    X = preprocess(raw_seqs, target_len=TARGET_LEN, use_derivatives=False, normalize=True)
    y = labels
    
    # -- Cross-validation (user-independent or user-dependent) --
    skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    accs = []
    
    for fold, (train_idx, test_idx) in enumerate(skf.split(X, y)):
        clf = DTWClassifier(window=0.1, constraint="sakoe_chiba", n_neighbors=1)
        clf.fit(X[train_idx], y[train_idx])
        acc = clf.score(X[test_idx], y[test_idx])
        accs.append(acc)
        print(f"Fold {fold+1}: accuracy = {acc:.4f}")
    
    print(f"\nMean accuracy: {np.mean(accs):.4f} ± {np.std(accs):.4f}")
```

### 5.3 FastDTW Variant (Approximate, Much Faster)

```python
from fastdtw import fastdtw
from scipy.spatial.distance import euclidean

def dtw_distance_fast(seq1: np.ndarray, seq2: np.ndarray, radius: int = 10) -> float:
    """
    Approximate DTW using FastDTW (O(n) time and space).
    seq1, seq2: (T, D) arrays
    """
    distance, _ = fastdtw(seq1, seq2, radius=radius, dist=euclidean)
    return distance
```

---

## 6. Parameters & Ablation Guide

| Parameter           | Type                 | Candidates                       | Effect                                                             |
| ------------------- | -------------------- | -------------------------------- | ------------------------------------------------------------------ |
| `target_len`        | Resampling length    | 32, **64**, 128                  | Controls time resolution; too short loses detail, too long is slow |
| `window`            | Sakoe-Chiba band     | None, 0.05, **0.10**, 0.20, 0.30 | Restricts alignment path; prevents degenerate mappings             |
| `constraint`        | Path type            | **`sakoe_chiba`**, `itakura`     | Shape of the accessible region in cost matrix                      |
| `use_derivatives`   | Feature augmentation | False, **True**                  | DDTW: uses shape rather than raw values; often improves accuracy   |
| `normalize`         | Normalization        | False, **True**                  | Removes amplitude differences between subjects                     |
| `metric` (distance) | Local cost           | **Euclidean**, Manhattan, Cosine | Inner distance between feature vectors at each step                |
| `n_neighbors`       | kNN                  | **1**, 3, 5                      | 1-NN is canonical; higher k can add robustness with more templates |

### Parameter Explanations

**`target_len` (Resampling length)**
DTW requires comparable sequence representations. Resampling to a fixed length $L$ normalizes temporal length before alignment. A larger $L$ preserves more temporal detail but increases computation. Rule of thumb: choose $L$ close to the median sequence length in your dataset.

**`window` (Sakoe-Chiba radius)**
The band radius $r$ restricts alignment to index pairs $(i,j)$ where $|i-j| \leq r$. A value of `None` yields unconstrained DTW, which may allow pathological alignments. Expressed as a fraction of `target_len` (e.g. `0.10` with `target_len=64` → radius of 6 frames). Typical ablation range: None, 5%, 10%, 20%, 30%.

**`use_derivatives` (DDTW)**
Replaces raw coordinates with first-order difference estimates. Makes the similarity measure sensitive to the *shape* of the trajectory rather than its absolute position in space — useful for translation-invariant gesture comparison.

**`normalize`**
Per-sequence z-score normalization removes between-subject differences in gesture amplitude. Critical for user-independent evaluation.

---

## 7. Limitations

- **Quadratic complexity:** $\mathcal{O}(n^2)$ per comparison; slow with long sequences or large template libraries. Mitigated by the Sakoe-Chiba band or FastDTW → [[Papers/Salvador2007|Salvador & Chan (2007)]].
- **No learning:** DTW does not update its representation based on training data — all training samples are used as templates. Contrast with [[07_BiLSTM|BiLSTM]] which learns a parameterised model.
- **High variance:** As a non-parametric method, 1-NN DTW has zero bias but potentially high variance — see [[Concepts/Bias_Variance_Regularisation|Bias-Variance Trade-off]].
- **Sensitive to outliers:** A single noisy frame can significantly affect the warping path → [[Workflows/Data_Preprocessing|Data Preprocessing]] (outlier removal, normalisation).
- **Template selection:** 1-NN DTW stores the entire training set. With more samples per class, accuracy generally improves but prediction slows.

---

## 8. References

> See `gesture_recognition.bib` for importable BibTeX entries.

- Sakoe, H., & Chiba, S. (1978). Dynamic programming algorithm optimization for spoken word recognition. *IEEE Transactions on Acoustics, Speech, and Signal Processing*, 26(1), 43–49. → [[Papers/Sakoe1978|Sakoe1978]]
- Berndt, D. J., & Clifford, J. (1994). Using dynamic time warping to find patterns in time series. *KDD Workshop*. → [[Papers/Berndt1994|Berndt1994]]
- Keogh, E. J., & Pazzani, M. J. (2001). Derivative dynamic time warping. *SIAM International Conference on Data Mining*. → [[Papers/Keogh2001_DDTW|Keogh2001_DDTW]]
- Salvador, S., & Chan, P. (2007). Toward accurate dynamic time warping in linear time and space. *Intelligent Data Analysis*, 11(5), 561–580. → [[Papers/Salvador2007|Salvador2007]]
- Senin, P. (2008). Dynamic time warping algorithm review. *Technical Report*, University of Hawaii. → [[Papers/Senin2008|Senin2008]]
- Ding, H. et al. (2008). Querying and mining of time series data. *VLDB Endowment*. → [[Papers/Ding2008|Ding2008]] *(justifies 1-NN DTW as best-in-class baseline)*
