---
title: "Data Preprocessing & Feature Engineering"
tags: [workflow, preprocessing, feature-engineering, normalisation, augmentation, practical]
created: 2026-05-22
---

# Data Preprocessing & Feature Engineering

> Transforms raw, messy data into clean, structured inputs ready for model training. The quality of preprocessing often determines model performance more than the choice of model.

---

## 1. Data Cleaning

### Missing Values
- **Drop:** If < 1–5% of samples are missing entirely — safest option
- **Impute (mean/median):** For small gaps in continuous features
- **Interpolate:** For time series — linear or cubic spline interpolation between valid frames
- **Forward/backward fill:** For sensor dropouts (carry last valid reading)

### Outlier Detection
- **Z-score:** Flag $|z| > 3$ as potential outliers
- **IQR method:** Flag $x < Q1 - 1.5 \cdot IQR$ or $x > Q3 + 1.5 \cdot IQR$
- **Context-aware:** In gesture sequences, a single anomalous frame may be labelling error vs. genuine fast motion — inspect manually before removing

### Duplicate / Labelling Errors
- Check exact duplicates (hash sequences)
- Review classes with very high within-class variance — may indicate labelling mistakes

---

## 2. Normalisation and Scaling

### Per-Feature Standardisation (Z-score)
$$x' = \frac{x - \mu}{\sigma}$$
Use when features have different scales. Compute $\mu, \sigma$ on training set only; apply same statistics to validation and test.

### Min-Max Normalisation
$$x' = \frac{x - x_\min}{x_\max - x_\min} \in [0,1]$$
Sensitive to outliers; use after removing them.

### Per-Sequence Normalisation (Gesture-specific)
- **Translation invariance:** Subtract the first frame or centroid from all frames
- **Scale invariance:** Divide by bounding-box diagonal or max displacement
- **Rotation invariance:** Align principal axis with a canonical direction (PCA or Kabsch) → [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]]

### Feature-wise vs. Global
- **Feature-wise:** Each coordinate axis ($x$, $y$, $z$) gets its own $\mu$, $\sigma$
- **Global:** Single statistics across all axes — preserves relative scale between axes (important when $z$ varies less than $x$)

---

## 3. Sequence Resampling

Many gesture algorithms require fixed-length sequences. Approaches:

| Method | When to use | Notes |
|---|---|---|
| **Linear interpolation** | Smooth trajectories | Adds frames between existing ones |
| **Linear subsampling** | Downsampling | Evenly pick every $k$th frame |
| **DTW resampling** | Matching to a template | Warps in time to align with reference |
| **Padding + masking** | Neural networks | Pad to max length; use attention mask |

The $3 Recognizer and $P Recognizer normalise to $N=32$ or $N=64$ resampled points.

---

## 4. Feature Engineering (Classical Methods)

For non-end-to-end methods (DTW, edit distance, SVM), hand-crafted features can improve performance:

### Kinematic Features
- **Velocity:** $v_t = (p_{t+1} - p_{t-1}) / 2\Delta t$ (finite difference)
- **Acceleration:** $a_t = v_{t+1} - v_{t-1}$
- **Curvature:** $\kappa = \|v \times a\| / \|v\|^3$
- **Speed:** $\|v_t\|_2$

### Shape Features
- **Bounding box aspect ratio**
- **Trajectory length** (total arc length)
- **Turning angle** between consecutive segments

### Frequency-Domain Features
- **FFT coefficients** of each coordinate time series
- **DCT** (Discrete Cosine Transform) — compact energy representation

### Codebook / Vector Quantisation
For edit distance methods: quantise feature vectors to a finite alphabet using k-means codebook → [[Concepts/Vector_Quantization|Vector Quantization]]

---

## 5. Data Augmentation

Augmentation artificially increases training set size by applying label-preserving transformations. Especially important for deep learning with small datasets.

### Geometric Augmentations (3D Gestures)
- **Random rotation:** Apply random rotation matrix $R$ to all frames ($\pm 15°$ around each axis)
- **Random scaling:** Scale coordinates by $s \sim \mathcal{U}(0.9, 1.1)$
- **Mirroring:** Reflect left-right (only if gesture is symmetric)
- **Translation jitter:** Add small random offset to all frames

### Temporal Augmentations
- **Time warping:** Random non-linear stretching/compressing of the time axis (DTW path–inspired)
- **Frame dropout:** Randomly drop 10–20% of frames, then interpolate
- **Speed variation:** Resample at $t' = t^\alpha$ for $\alpha \sim \mathcal{U}(0.8, 1.2)$

### Noise Augmentation
- **Gaussian noise:** Add $\varepsilon \sim \mathcal{N}(0, \sigma^2)$ to each coordinate ($\sigma \approx 0.01$ after normalisation)
- **Sensor noise models:** Accelerometer drift, gyroscope bias — model realistically if known

### Augmentation Principle
Only augment the **training set**. Validation and test sets remain unaugmented — they represent the real test distribution.

---

## 6. Train / Validation / Test Split

| Protocol | Description | Use when |
|---|---|---|
| **Random split** | Shuffle all samples, split 70/15/15 | IID data, no temporal/user structure |
| **User-independent CV** | Leave-one-user-out (LOUO) | Gesture recognition — model must generalise to new users |
| **User-dependent CV** | Leave-one-sample-out (LOSO) | Per-user personalised models |
| **Temporal split** | Train on older data, test on newer | Time-series with concept drift |

> For the Huang dataset: always use user-independent CV for the main reported result.

See → [[Concepts/Cross_Validation|Cross-Validation Protocols]]

---

## Connections

- Full pipeline → [[Workflows/ML_Pipeline|ML Pipeline]]
- Cross-validation protocols → [[Concepts/Cross_Validation|Cross-Validation]]
- Normalisation → [[Foundations/Linear_Algebra_for_ML|Linear Algebra for ML]] (norms, SVD)
- Codebook → [[Concepts/Vector_Quantization|Vector Quantization]]
- Kabsch rotation → [[Concepts/Kabsch_Algorithm|Kabsch Algorithm]]
