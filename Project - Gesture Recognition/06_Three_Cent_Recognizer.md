# Three Cent ($3) Gesture Recognizer

---
tags:
  - gesture-recognition
  - sota
  - template-matching
  - 3d
  - dollar-family
  - rotation-invariance
  - sequence-classification
---

> **Role in study:** SOTA template-matching method  
> **Key papers:** [[Papers/Wobbrock2007|Wobbrock et al. (2007)]], [[Papers/Kratz2010|Kratz & Rohs (2010)]], [[Papers/Kratz2011_Protractor3D|Kratz & Rohs (2011)]], [[Papers/Li2010_Protractor|Li & Kosecka (2010)]], [[Papers/Vatavu2012_P|Vatavu et al. (2012)]]  
> **Concepts:** [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]], [[Concepts/kNN_Classifier|k-NN Classifier]], [[Concepts/Cross_Validation|Cross-Validation Protocol]], [[Concepts/Dimensionality_Reduction|Dimensionality Reduction (PCA alignment)]], [[Concepts/Supervised_Learning_Framework|Supervised Learning Framework]]  
> **Foundations:** [[Foundations/Linear_Algebra_for_ML|Linear Algebra (SVD, rotation)]]  
> **Workflows:** [[Workflows/Data_Preprocessing|Data Preprocessing]], [[Workflows/Model_Evaluation|Model Evaluation]]  
> **Related methods:** [[04_DTW_kNN|DTW]], [[05_Edit_Distance_kNN|Edit Distance]], [[07_BiLSTM|BiLSTM]]

---

## 1. Conceptual Overview

The **$3 Recognizer** (Kratz & Rohs, 2010) is a 3D extension of the iconic **$1 Recognizer** (Wobbrock, Wilson & Li, 2007), designed for gesture recognition on devices equipped with 3D acceleration sensors. Its design philosophy is captured in its name — minimal cost, minimal complexity, maximum deployability.

The $1 Recognizer was a landmark paper in HCI: by reducing gesture recognition to a small set of geometric transformations followed by template matching, Wobbrock et al. demonstrated that it was possible to build accurate, trainable recognizers with fewer than 100 lines of code — no machine learning frameworks, no feature engineering, no training phase beyond storing examples.

The $3 Recognizer adapts this approach to **3D trajectory data** (accelerometer, skeleton, depth sensors), making it directly applicable to hand gesture recognition from inertial or skeletal inputs.

---

## 2. The $1 Recognizer: Foundations (Wobbrock et al., 2007)

Understanding $3 requires understanding $1.

### 2.1 Core Idea

$1 is a **nearest-neighbor template matcher** operating in 2D. Its key insight is that after applying a small set of geometric normalizations, gestures become comparable via simple point-by-point Euclidean distance — no DTW, no HMMs, no feature engineering.

### 2.2 Algorithm (2D)

Given a candidate gesture (a sequence of 2D points):

**Step 1 — Resample:**
Resample the stroke to exactly $N$ evenly-spaced points (default $N=64$). This normalizes for temporal variability.

$$p_i = \text{lerp}\left(p_{\lfloor t_i \rfloor}, p_{\lceil t_i \rceil}\right), \quad i = 1, \ldots, N$$

**Step 2 — Rotate to indicative angle:**
Compute the angle $\varphi$ from the centroid to the first point, then rotate the entire gesture by $-\varphi$ so all gestures begin at the same indicative angle.

**Step 3 — Scale to reference square:**
Scale the bounding box of the gesture to fit inside a $[-1,1]^2$ square. This normalizes for gesture size.

**Step 4 — Translate to origin:**
Translate the centroid to $(0, 0)$.

**Step 5 — Match:**
For each template, compute the average Euclidean distance between corresponding resampled points. The nearest template wins.

The Golden Section Search (GSS) finds the optimal rotation angle within $[-45°, +45°]$ for rotation-invariant matching.

---

## 3. The $3 Recognizer: 3D Extension (Kratz & Rohs, 2010)

### 3.1 From 2D to 3D: The Core Adaptation

The $3 Recognizer extends $1's pipeline to 3D point sequences $(x, y, z)$. The key changes are:

**Resampling:** Identical to $1 but in 3D, using the 3D arc-length:

$$d_i = \|p_i - p_{i-1}\|_2 \quad \text{in } \mathbb{R}^3$$

**Rotation normalization:** In 3D, rotation normalization is more complex. Kratz & Rohs use a two-stage approach:
1. **Align principal axis:** Use PCA on the point cloud to find the dominant direction. Rotate so the first principal component aligns with a reference axis (e.g., the $x$-axis).
2. **Fine-tune rotation:** Apply a 1D Golden Section Search around the dominant axis to find the best rotational alignment with each template.

**Scaling:** Scale the 3D bounding box uniformly to a reference cube of side 2.

**Translation:** Center the gesture at the origin.

**Distance metric:** Average point-cloud distance after alignment:

$$\text{dist}(G, T) = \frac{1}{N} \sum_{i=1}^{N} \|g_i - t_i\|_2$$

where $G = (g_1, \ldots, g_N)$ and $T = (t_1, \ldots, t_N)$ are the resampled candidate and template.

### 3.2 Protractor3D (Kratz & Rohs, 2011): Closed-Form Rotation

A direct follow-up by the same authors replaces the iterative Golden Section Search with a **closed-form optimal rotation** using singular value decomposition (SVD). This is dramatically faster (no iterative search) and more accurate. The optimal rotation $R^*$ between two point clouds minimizes:

$$R^* = \underset{R \in SO(3)}{\arg\min} \sum_{i=1}^N \|R g_i - t_i\|_2^2$$

This is the **Kabsch algorithm** (also known as the Wahba problem):
1. Compute cross-covariance matrix $H = G^\top T$
2. SVD: $H = U \Sigma V^\top$
3. $R^* = V U^\top$ (with sign correction to ensure $\det(R^*) = +1$)

### 3.3 Classification Score

The $1/$3 score is normalized to $[0, 1]$:

$$\text{score} = 1 - \frac{\text{dist}(G, T)}{\frac{1}{2}\sqrt{3} \cdot \text{size}}$$

where $\text{size}$ is the reference box size (e.g., $2\sqrt{3}$ for a unit cube).

---

## 4. Recognition Pipeline for Gesture Recognition

```
Input: 3D gesture trajectory (T, 3) or (T, D) for multi-joint
           │
           ▼
    Resample to N points (arc-length parameterization)
           │
           ▼
    Apply 3D geometric normalization:
      - PCA alignment
      - Scale to unit cube
      - Translate to origin
           │
           ▼
    For each (class, template):
      Compute optimal rotation (GSS or Kabsch/SVD)
      Compute average point-cloud distance
           │
           ▼
    Return class with minimum distance (1-NN)
```

---

## 5. Applicability to Multi-Joint Skeletons

While originally designed for single-trajectory inputs (e.g., one accelerometer), $3 can be extended to multi-joint skeleton data by **concatenating** joint coordinates into a single high-dimensional vector per frame:

$$p_t = [x_t^{(1)}, y_t^{(1)}, z_t^{(1)}, \ldots, x_t^{(J)}, y_t^{(J)}, z_t^{(J)}] \in \mathbb{R}^{3J}$$

This turns the problem into high-dimensional template matching. PCA-based alignment then operates on this $3J$-dimensional space.

Alternatively, per-joint distances can be aggregated:

$$\text{dist}(G, T) = \frac{1}{N} \sum_{i=1}^N \frac{1}{J} \sum_{j=1}^J \|g_i^{(j)} - t_i^{(j)}\|_2$$

---

## 6. Model: Python Implementation

### 6.1 Dependencies

```bash
pip install numpy scipy scikit-learn
```

> **Repository references:**
> - Original $1 JavaScript implementation: https://depts.washington.edu/acelab/proj/dollar/index.html
> - Python $1 port reference: https://github.com/niklasvh/dollar-one-gesture-recognizer
> - This implementation follows Kratz & Rohs (2010) adapted for skeleton data

### 6.2 Full Implementation

```python
import numpy as np
from scipy.spatial.transform import Rotation
from sklearn.metrics import accuracy_score
from typing import Optional


# ──────────────────────────────────────────────────────
# 1. Geometric Utilities
# ──────────────────────────────────────────────────────

def resample_3d(points: np.ndarray, N: int) -> np.ndarray:
    """
    Resample a (T, D) trajectory to exactly N equidistant points
    using arc-length parameterization (works in any dimension D).
    
    Args:
        points: Input trajectory (T, D)
        N: Target number of points
    Returns:
        Resampled trajectory (N, D)
    """
    T, D = points.shape
    # Cumulative arc lengths
    diffs   = np.diff(points, axis=0)                # (T-1, D)
    seg_len = np.linalg.norm(diffs, axis=1)           # (T-1,)
    cum_len = np.concatenate([[0], np.cumsum(seg_len)])  # (T,)
    total   = cum_len[-1]
    
    # Target arc positions
    target  = np.linspace(0, total, N)
    
    # Interpolate each dimension independently
    resampled = np.stack(
        [np.interp(target, cum_len, points[:, d]) for d in range(D)],
        axis=1
    )  # (N, D)
    return resampled


def scale_to_cube(points: np.ndarray, size: float = 2.0) -> np.ndarray:
    """Scale bounding box to [-size/2, size/2]^D uniformly."""
    mins = points.min(axis=0)
    maxs = points.max(axis=0)
    ranges = maxs - mins
    max_range = ranges.max() + 1e-8
    center = (mins + maxs) / 2
    return (points - center) / max_range * size


def translate_to_origin(points: np.ndarray) -> np.ndarray:
    """Translate centroid to origin."""
    return points - points.mean(axis=0, keepdims=True)


def pca_align(points: np.ndarray) -> np.ndarray:
    """
    Rotate points so that the first principal component aligns with the x-axis.
    Works for any dimensionality D but is most natural for 3D.
    """
    cov = np.cov(points.T)
    eigvals, eigvecs = np.linalg.eigh(cov)  # eigvecs: columns are eigenvectors
    # Sort by descending eigenvalue
    idx = np.argsort(eigvals)[::-1]
    R   = eigvecs[:, idx].T  # Rotation matrix: rows are principal components
    return points @ R.T


def kabsch_rotation(P: np.ndarray, Q: np.ndarray) -> np.ndarray:
    """
    Compute the optimal 3D rotation R* that minimizes sum ||R P_i - Q_i||^2.
    Implements the Kabsch algorithm (Protractor3D approach).
    
    Args:
        P, Q: (N, D) point matrices to align
    Returns:
        R: (D, D) rotation matrix such that R @ p ≈ q
    """
    # Cross-covariance
    H = P.T @ Q  # (D, D)
    U, S, Vt = np.linalg.svd(H)
    # Correct for reflection (ensure proper rotation)
    d = np.linalg.det(Vt.T @ U.T)
    D = np.diag([1.0] * (P.shape[1] - 1) + [d])
    R = Vt.T @ D @ U.T
    return R


def golden_section_search_rotation(candidate: np.ndarray,
                                    template: np.ndarray,
                                    n_iters: int = 10) -> float:
    """
    Find optimal rotation angle (around z-axis) via Golden Section Search.
    Used for 2D/simple 3D cases. Returns the minimum average distance.
    
    Args:
        candidate, template: (N, D) resampled and normalized point sequences
    Returns:
        Minimum path distance after optimal rotation
    """
    phi = (1 + np.sqrt(5)) / 2  # Golden ratio
    a, b = -np.pi / 4, np.pi / 4
    
    def dist_at_angle(theta: float) -> float:
        c, s = np.cos(theta), np.sin(theta)
        R = np.array([[c, -s], [s, c]])  # 2D rotation
        rotated = (candidate[:, :2] @ R.T)
        return np.mean(np.linalg.norm(rotated - template[:, :2], axis=1))
    
    x1 = b - (b - a) / phi
    x2 = a + (b - a) / phi
    f1, f2 = dist_at_angle(x1), dist_at_angle(x2)
    
    for _ in range(n_iters):
        if f1 < f2:
            b, x2, f2 = x2, x1, f1
            x1 = b - (b - a) / phi
            f1 = dist_at_angle(x1)
        else:
            a, x1, f1 = x1, x2, f2
            x2 = a + (b - a) / phi
            f2 = dist_at_angle(x2)
    
    return min(f1, f2)


# ──────────────────────────────────────────────────────
# 2. Three-Cent Recognizer
# ──────────────────────────────────────────────────────

class ThreeCentRecognizer:
    """
    $3 Gesture Recognizer (Kratz & Rohs, 2010) adapted for skeleton/3D data.
    Uses template matching with geometric normalization and Kabsch rotation.
    """
    
    def __init__(self,
                 N: int = 64,
                 rotation_method: str = "kabsch",
                 scale: bool = True,
                 pca_align_flag: bool = True,
                 box_size: float = 2.0):
        """
        Args:
            N: Number of resampling points
            rotation_method: "kabsch" (SVD, optimal) or "gss" (iterative search, $3 original)
            scale: Whether to scale to unit bounding box
            pca_align_flag: Whether to apply PCA alignment before matching
            box_size: Reference box size for scaling
        """
        self.N                 = N
        self.rotation_method   = rotation_method
        self.scale             = scale
        self.pca_align_flag    = pca_align_flag
        self.box_size          = box_size
        
        self.templates_: list[np.ndarray] = []   # Normalized templates
        self.labels_:    list[int]         = []

    def _normalize(self, seq: np.ndarray) -> np.ndarray:
        """Apply full normalization pipeline to a sequence."""
        seq = resample_3d(seq, self.N)
        seq = translate_to_origin(seq)
        if self.pca_align_flag:
            seq = pca_align(seq)
        if self.scale:
            seq = scale_to_cube(seq, self.box_size)
        return seq

    def fit(self, sequences: list[np.ndarray], labels: np.ndarray) -> "ThreeCentRecognizer":
        """
        Store normalized templates from training data.
        Args:
            sequences: List of (T_i, D) arrays
            labels: Class labels (N,)
        """
        self.templates_ = []
        self.labels_    = []
        for seq, lbl in zip(sequences, labels):
            norm = self._normalize(seq)
            self.templates_.append(norm)
            self.labels_.append(int(lbl))
        return self

    def _distance(self, candidate: np.ndarray, template: np.ndarray) -> float:
        """Compute distance between two normalized (N, D) sequences."""
        if self.rotation_method == "kabsch":
            R = kabsch_rotation(candidate, template)
            rotated = candidate @ R.T
            return float(np.mean(np.linalg.norm(rotated - template, axis=1)))
        elif self.rotation_method == "gss":
            # Original $3 approach: 1D GSS around z-axis (works best for 3D accel)
            return golden_section_search_rotation(candidate, template)
        elif self.rotation_method == "none":
            # No rotation alignment (use after PCA alignment)
            return float(np.mean(np.linalg.norm(candidate - template, axis=1)))
        else:
            raise ValueError(f"Unknown rotation_method: {self.rotation_method}")

    def predict(self, sequences: list[np.ndarray]) -> np.ndarray:
        predictions = []
        for seq in sequences:
            candidate = self._normalize(seq)
            
            best_dist = float("inf")
            best_lbl  = -1
            
            for tmpl, lbl in zip(self.templates_, self.labels_):
                d = self._distance(candidate, tmpl)
                if d < best_dist:
                    best_dist = d
                    best_lbl  = lbl
            
            predictions.append(best_lbl)
        return np.array(predictions)

    def score(self, sequences: list[np.ndarray], labels: np.ndarray) -> float:
        return accuracy_score(labels, self.predict(sequences))


# ──────────────────────────────────────────────────────
# 3. Usage Example
# ──────────────────────────────────────────────────────

if __name__ == "__main__":
    from sklearn.model_selection import StratifiedKFold
    
    np.random.seed(42)
    N_CLASSES, N_PER_CLASS = 14, 5
    N = N_CLASSES * N_PER_CLASS
    
    # Simulate 3D hand trajectories (21 joints × 3 = 63 features)
    raw_seqs = [np.random.randn(np.random.randint(40, 80), 63) for _ in range(N)]
    labels   = np.repeat(np.arange(N_CLASSES), N_PER_CLASS)
    
    skf  = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    accs = []
    
    for fold, (train_idx, test_idx) in enumerate(skf.split(raw_seqs, labels)):
        train_seqs = [raw_seqs[i] for i in train_idx]
        test_seqs  = [raw_seqs[i] for i in test_idx]
        
        rec = ThreeCentRecognizer(
            N=64,
            rotation_method="kabsch",
            scale=True,
            pca_align_flag=True,
        )
        rec.fit(train_seqs, labels[train_idx])
        acc = rec.score(test_seqs, labels[test_idx])
        accs.append(acc)
        print(f"Fold {fold+1}: {acc:.4f}")
    
    print(f"\nMean: {np.mean(accs):.4f} ± {np.std(accs):.4f}")
```

---

## 7. Parameters & Ablation Guide

| Parameter | Candidates | Default | Effect |
|---|---|---|---|
| `N` (Resampling points) | 32, **64**, 128 | 64 | Central parameter; controls temporal resolution of templates |
| `rotation_method` | **`kabsch`**, `gss`, `none` | `kabsch` | How rotational alignment is performed during matching |
| `pca_align_flag` | **True**, False | True | PCA pre-alignment; removes dominant rotation ambiguity |
| `scale` | **True**, False | True | Normalizes gesture size; critical for user-independent eval |
| `n_templates_per_class` | 1, **3**, 5, all | 3 | Number of stored templates per gesture class |
| `template_selection` | first, **random**, centroid | random | How templates are selected when multiple per class are stored |

### Parameter Explanations

**`N` (Number of resampling points)**
The single most important parameter. The $1/$3 family achieves temporal invariance through fixed-length resampling, not warping. A small $N$ (32) compresses the gesture aggressively — fast but potentially lossy. A large $N$ (128) preserves more temporal detail but increases the matching cost.

**`rotation_method`**
- `kabsch`: Uses SVD to find the globally optimal 3D rotation (Protractor3D). Exact, fast, recommended for skeleton data.
- `gss`: The original $3 approach — iterative Golden Section Search for the best 1D rotation angle. Works well for accelerometer gestures with a dominant rotation axis.
- `none`: No rotation alignment (relies entirely on PCA pre-alignment). Fastest but may fail for gestures with significant orientation variability.

**`pca_align_flag`**
PCA alignment rotates the gesture so its principal axis of variation aligns with a canonical direction. This resolves most orientation ambiguity without per-template search. Disable this to study the effect of the full rotation search.

**`scale`**
Scaling to a unit bounding box normalizes gesture amplitude. Disabling it makes the recognizer sensitive to gesture size, which may hurt user-independent performance (different hand sizes, distances from sensor).

**`n_templates_per_class` / `template_selection`**
Multiple templates per class increase recognition rate at the cost of more comparisons. With all training samples as templates, $3 becomes a full 1-NN classifier. The centroid template (computed as the mean of normalized templates) is a compact alternative.

---

## 8. Complexity Analysis

| Operation | Complexity |
|---|---|
| Resampling | $\mathcal{O}(T + N)$ |
| PCA alignment | $\mathcal{O}(N \cdot D^2)$ |
| Kabsch rotation (per template) | $\mathcal{O}(N \cdot D + D^3)$ |
| Distance computation (per template) | $\mathcal{O}(N \cdot D)$ |
| Full recognition (K templates) | $\mathcal{O}(K \cdot (N \cdot D + D^3))$ |

For hand skeleton data ($D = 63$, $N = 64$, $K \sim 70$ templates): approximately $10^5$ operations per recognition — extremely fast.

---

## 9. Limitations

- **No learning:** $3 does not update its representation. With few templates per class, it may miss intra-class variation.
- **Rotation ambiguity in $D > 3$:** For high-dimensional inputs (multi-joint flattened), PCA + Kabsch may not find a meaningful rotation. Consider per-joint normalization instead.
- **Shape vs. dynamics:** $3 captures spatial trajectory shape but ignores velocity or acceleration profiles. Two gestures with identical paths but different temporal dynamics will receive the same representation.

---

## 10. References

> See `gesture_recognition.bib` for importable BibTeX entries.

- Wobbrock, J. O., Wilson, A. D., & Li, Y. (2007). Gestures without libraries, toolkits or training: a $1 recognizer for user interface prototypes. *UIST 2007*, 159–168. → [[Papers/Wobbrock2007|Wobbrock2007]] *(required citation per guidelines)*
- Kratz, S., & Rohs, M. (2010). A $3 gesture recognizer. *IUI 2010*. → [[Papers/Kratz2010|Kratz2010]]
- Kratz, S., & Rohs, M. (2011). Protractor3D: closed-form rotation for 3D gestures. *IUI 2011*, 371–374. → [[Papers/Kratz2011_Protractor3D|Kratz2011_Protractor3D]]
- Vatavu, R.-D., Anthony, L., & Wobbrock, J. O. (2012). $P recognizer. *ICMI 2012*. → [[Papers/Vatavu2012_P|Vatavu2012_P]]
- Li, Y. (2010). Protractor: fast and accurate 2D gesture recognizer. *CHI 2010*. → [[Papers/Li2010_Protractor|Li2010_Protractor]]
