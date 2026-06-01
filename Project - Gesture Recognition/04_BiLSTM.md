# Bidirectional LSTM (BiLSTM) for Gesture Recognition

---
tags:
  - gesture-recognition
  - sota
  - deep-learning
  - lstm
  - bilstm
  - rnn
  - sequence-modeling
  - sequence-classification
---

> **Role in study:** SOTA deep learning method  
> **Key papers:** [[Papers/Hochreiter1997|Hochreiter & Schmidhuber (1997)]], [[Papers/Schuster1997|Schuster & Paliwal (1997)]], [[Papers/Graves2005_BiLSTM|Graves & Schmidhuber (2005)]], [[Papers/Zhang2021_SkeletonBiLSTM|Zhang et al. (2021)]], [[Papers/Gers2000|Gers et al. (2000)]] (forget gate), [[Papers/Goodfellow2016|Goodfellow et al. (2016) Ch.10]], [[Papers/Srivastava2014_Dropout|Srivastava et al. (2014)]] (dropout), [[Papers/Ioffe2015_BatchNorm|Ioffe & Szegedy (2015)]] (batch/layer norm)  
> **Concepts:** [[Concepts/Backpropagation|Backpropagation]] · [[Concepts/Neural_Networks_MLP|Neural Networks & MLP]] · [[Concepts/Bias_Variance_Regularisation|Bias-Variance & Regularisation]] · [[Concepts/Cross_Validation|Cross-Validation Protocol]] · [[Concepts/Attention_Transformer|Attention (alternative to pooling)]]  
> **Foundations:** [[Foundations/Optimisation_Theory|Optimisation Theory (Adam, gradient clipping)]] · [[Foundations/Probability_Theory|Probability Theory (dropout as Bernoulli noise)]]  
> **Optimiser:** [[Papers/Kingma2014_Adam|Adam (Kingma & Ba, 2014)]]  
> **Workflows:** [[Workflows/Data_Preprocessing|Data Preprocessing]] · [[Workflows/Model_Evaluation|Model Evaluation]] · [[Workflows/Hyperparameter_Tuning|Hyperparameter Tuning]]  
> **HMM alternative:** [[Papers/Fink2014_MarkovModels|Fink (2014)]] · [[Papers/Rabiner1993_SpeechRecognition|Rabiner & Juang (1993)]]  
> **Related methods:** [[01_DTW_Dynamic_Time_Warping|DTW]], [[02_Edit_Distance|Edit Distance]], [[03_Three_Cent_Recognizer|$3 Recognizer]]

---

## 1. Conceptual Overview

The **Bidirectional Long Short-Term Memory (BiLSTM)** network is a deep learning architecture that processes sequential data in both forward and backward temporal directions, combining the outputs to form a richer, context-aware representation at each time step.

In gesture recognition, a gesture is a time series of feature vectors (e.g., 3D joint positions). BiLSTM naturally handles:
- **Variable-length sequences** (via padding and masking)
- **Long-range temporal dependencies** (the memory cell mitigates vanishing gradients)
- **Bidirectional context** (the end of a gesture informs the interpretation of its beginning — important for distinguishing similar gesture pairs)

BiLSTM-based architectures consistently outperform classical methods (DTW, edit distance, $3) on benchmark datasets such as DHG-14/28, SHREC'17, and ChaLearn LAP when sufficient training data is available.

---

## 2. Theoretical Foundations

### 2.1 The Vanishing Gradient Problem in RNNs

Standard Recurrent Neural Networks (RNNs) process sequences step by step, maintaining a hidden state $h_t$. The **vanishing gradient problem** → see [[Concepts/Backpropagation|Backpropagation]]:

$$h_t = \tanh(W_h h_{t-1} + W_x x_t + b)$$

The gradient of the loss w.r.t. an early weight involves the product of many Jacobians:

$$\frac{\partial \mathcal{L}}{\partial W} \propto \prod_{t=k}^{T} \frac{\partial h_t}{\partial h_{t-1}} = \prod_{t=k}^{T} W_h \text{diag}(\tanh'(\cdot))$$

If $\|W_h\| < 1$, this product converges to zero — the **vanishing gradient** problem (Hochreiter, 1991). For gesture sequences of 40–120 frames, this makes it impossible for vanilla RNNs to learn long-range temporal patterns.

### 2.2 Long Short-Term Memory (Hochreiter & Schmidhuber, 1997)

LSTM introduces a **cell state** $C_t$ that flows through time with only multiplicative (element-wise) interactions — a "highway" that prevents gradient vanishing. Three **gates** control information flow:

**Forget gate** — what to discard from the previous cell state:
$$f_t = \sigma(W_f [h_{t-1}, x_t] + b_f)$$

**Input gate** — what new information to write:
$$i_t = \sigma(W_i [h_{t-1}, x_t] + b_i)$$
$$\tilde{C}_t = \tanh(W_C [h_{t-1}, x_t] + b_C)$$

**Cell state update:**
$$C_t = f_t \odot C_{t-1} + i_t \odot \tilde{C}_t$$

**Output gate** — what to expose as the hidden state:
$$o_t = \sigma(W_o [h_{t-1}, x_t] + b_o)$$
$$h_t = o_t \odot \tanh(C_t)$$

where $\sigma$ is the sigmoid function, $\odot$ is element-wise multiplication, and $[h_{t-1}, x_t]$ denotes concatenation.

**Key property:** The gradient of $C_t$ w.r.t. $C_{t-1}$ is $f_t$ — a learned, input-dependent gate value in $(0,1)$. This replaces the fixed spectral constraint and allows the network to decide when to preserve or reset memory.

### 2.3 Bidirectional LSTM (Schuster & Paliwal, 1997)

A BiLSTM runs two separate LSTMs over the same sequence:

- **Forward LSTM:** processes $x_1, x_2, \ldots, x_T$ → produces $\overrightarrow{h}_1, \ldots, \overrightarrow{h}_T$
- **Backward LSTM:** processes $x_T, x_{T-1}, \ldots, x_1$ → produces $\overleftarrow{h}_T, \ldots, \overleftarrow{h}_1$

The output at each time step is the concatenation:

$$h_t = [\overrightarrow{h}_t;\ \overleftarrow{h}_t] \in \mathbb{R}^{2H}$$

where $H$ is the hidden size of each directional LSTM.

**Why bidirectionality helps for gesture recognition:**
The end of a gesture (the return to rest position, the final hand pose) is highly discriminative and informs the interpretation of earlier frames. A bidirectional architecture can condition the representation of frame $t$ on both past and future context within the gesture window.

Graves, Fernández & Schmidhuber (2005) first demonstrated the empirical superiority of BiLSTM over forward-only LSTM on sequential classification tasks.

---

## 3. Architecture for Gesture Classification

### 3.1 Input Representation

For 3D hand skeleton data with $J$ joints:

$$x_t \in \mathbb{R}^{3J} \quad \text{(raw positions)}$$

Optionally augmented with velocity and acceleration:

$$x_t = [\underbrace{p_t}_{3J},\ \underbrace{p_t - p_{t-1}}_{3J},\ \underbrace{p_t - 2p_{t-1} + p_{t-2}}_{3J}] \in \mathbb{R}^{9J}$$

### 3.2 Standard BiLSTM Classification Architecture

```
Input:  (B, T, D)   — batch × timesteps × features
   │
   ▼
[Optional] Linear input projection: (B, T, D) → (B, T, H)
   │
   ▼
BiLSTM Layer 1:  (B, T, H) → (B, T, 2H)
   │    Dropout(p)
   ▼
BiLSTM Layer 2:  (B, T, 2H) → (B, T, 2H)
   │    Dropout(p)
   ▼
[Pooling] Global average or last-timestep: (B, T, 2H) → (B, 2H)
   │
   ▼
Fully Connected:  (B, 2H) → (B, num_classes)
   │
   ▼
Softmax → Class probabilities
```

### 3.3 Sequence Aggregation Strategies

| Strategy | Formula | Notes |
|---|---|---|
| **Last timestep** | $h_T$ (forward) | Simple; misses backward context at last step |
| **Bidirectional last** | $[\overrightarrow{h}_T;\ \overleftarrow{h}_1]$ | Forward end + backward end |
| **Global average pooling** | $\frac{1}{T}\sum_t h_t$ | Robust; averages all timestep representations |
| **Attention** | $\sum_t \alpha_t h_t$ where $\alpha_t = \text{softmax}(e_t)$ | Learned weighted average; most expressive |

Global average pooling is recommended as a strong default for gesture classification.

---

## 4. Model: PyTorch Implementation

### 4.1 Dependencies

```bash
pip install torch numpy scikit-learn
```

> **Reference architectures:**
> - Skeleton-based gesture classification with BiLSTM: Zhang et al. (2021) — *Applied Sciences*, 11(10), 4689
> - Graves & Schmidhuber BiLSTM: original framework in Graves et al. (2005)

### 4.2 Full Implementation

```python
import numpy as np
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
from torch.nn.utils.rnn import pad_sequence, pack_padded_sequence, pad_packed_sequence
from sklearn.metrics import accuracy_score, classification_report
from typing import Optional


# ──────────────────────────────────────────────────────
# 1. Dataset
# ──────────────────────────────────────────────────────

class GestureDataset(Dataset):
    """
    Dataset for variable-length gesture sequences.
    Handles padding and length tracking for PackedSequence.
    """
    
    def __init__(self,
                 sequences: list[np.ndarray],
                 labels: np.ndarray,
                 max_len: Optional[int] = None):
        """
        Args:
            sequences: List of (T_i, D) float32 arrays
            labels: Class indices (N,)
            max_len: If set, truncate sequences to this length
        """
        self.labels = torch.tensor(labels, dtype=torch.long)
        self.lengths = []
        self.sequences = []
        
        for seq in sequences:
            if max_len is not None:
                seq = seq[:max_len]
            t = torch.tensor(seq, dtype=torch.float32)
            self.sequences.append(t)
            self.lengths.append(len(seq))
    
    def __len__(self):
        return len(self.sequences)
    
    def __getitem__(self, idx):
        return self.sequences[idx], self.labels[idx], self.lengths[idx]


def collate_fn(batch):
    """Collate with zero-padding for variable-length sequences."""
    seqs, labels, lengths = zip(*batch)
    # Sort by descending length (required for pack_padded_sequence)
    order   = sorted(range(len(lengths)), key=lambda i: lengths[i], reverse=True)
    seqs    = [seqs[i]   for i in order]
    labels  = torch.stack([labels[i]  for i in order])
    lengths = [lengths[i] for i in order]
    
    padded  = pad_sequence(seqs, batch_first=True)  # (B, T_max, D)
    return padded, labels, torch.tensor(lengths, dtype=torch.long)


# ──────────────────────────────────────────────────────
# 2. Model Architecture
# ──────────────────────────────────────────────────────

class BiLSTMGestureClassifier(nn.Module):
    """
    Multi-layer BiLSTM classifier for gesture recognition.
    
    Architecture:
        [Optional input projection]
        → N × BiLSTM layers with dropout
        → Global average pooling over time
        → Fully connected classifier
    """
    
    def __init__(self,
                 input_dim: int,
                 hidden_size: int = 128,
                 num_layers: int = 2,
                 num_classes: int = 14,
                 dropout: float = 0.3,
                 use_input_proj: bool = False,
                 proj_dim: int = 64,
                 pooling: str = "mean"):
        """
        Args:
            input_dim: Feature dimensionality D (e.g., 63 for 21 joints × 3)
            hidden_size: Hidden units per direction in each LSTM layer
            num_layers: Number of stacked BiLSTM layers
            num_classes: Number of gesture classes
            dropout: Dropout probability between layers (0 = no dropout)
            use_input_proj: Add a linear projection before LSTM
            proj_dim: Projection dimension (only if use_input_proj=True)
            pooling: "mean" (global avg pooling) | "last" | "attention"
        """
        super().__init__()
        self.hidden_size = hidden_size
        self.num_layers  = num_layers
        self.pooling     = pooling
        
        # Optional input projection
        lstm_input_dim = input_dim
        if use_input_proj:
            self.input_proj = nn.Sequential(
                nn.Linear(input_dim, proj_dim),
                nn.ReLU(),
            )
            lstm_input_dim = proj_dim
        else:
            self.input_proj = None
        
        # BiLSTM stack
        # Note: PyTorch's nn.LSTM with bidirectional=True handles the two directions
        self.lstm = nn.LSTM(
            input_size=lstm_input_dim,
            hidden_size=hidden_size,
            num_layers=num_layers,
            batch_first=True,
            bidirectional=True,
            dropout=dropout if num_layers > 1 else 0.0,
        )
        
        # Post-LSTM dropout
        self.dropout = nn.Dropout(dropout)
        
        # Attention (if used)
        if pooling == "attention":
            self.attention = nn.Linear(hidden_size * 2, 1)
        
        # Classifier head
        self.fc = nn.Linear(hidden_size * 2, num_classes)
    
    def forward(self, x: torch.Tensor, lengths: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x: Padded input (B, T, D)
            lengths: Actual sequence lengths (B,)
        Returns:
            logits: (B, num_classes)
        """
        # Input projection
        if self.input_proj is not None:
            x = self.input_proj(x)  # (B, T, proj_dim)
        
        # Pack sequences (avoids computation on padding)
        packed = pack_padded_sequence(x, lengths.cpu(), batch_first=True,
                                       enforce_sorted=True)
        
        # BiLSTM forward pass
        out_packed, _ = self.lstm(packed)
        
        # Unpack
        out, _ = pad_packed_sequence(out_packed, batch_first=True)  # (B, T, 2H)
        
        # Pooling over time
        if self.pooling == "mean":
            # Mask out padding before averaging
            mask = torch.arange(out.size(1), device=out.device)[None, :] < lengths[:, None]
            out  = out * mask.unsqueeze(-1).float()
            pooled = out.sum(dim=1) / lengths.float().unsqueeze(-1)  # (B, 2H)
        
        elif self.pooling == "last":
            # Use [forward h_T ; backward h_1] (natural BiLSTM summary)
            fwd = out[range(len(lengths)), lengths - 1, :self.hidden_size]  # (B, H)
            bwd = out[:, 0, self.hidden_size:]                               # (B, H)
            pooled = torch.cat([fwd, bwd], dim=-1)                           # (B, 2H)
        
        elif self.pooling == "attention":
            # Learned attention weights
            scores  = self.attention(out).squeeze(-1)  # (B, T)
            mask    = torch.arange(out.size(1), device=out.device)[None, :] >= lengths[:, None]
            scores  = scores.masked_fill(mask, float("-inf"))
            weights = torch.softmax(scores, dim=1).unsqueeze(-1)   # (B, T, 1)
            pooled  = (out * weights).sum(dim=1)                   # (B, 2H)
        
        else:
            raise ValueError(f"Unknown pooling: {self.pooling}")
        
        pooled = self.dropout(pooled)
        return self.fc(pooled)  # (B, num_classes)


# ──────────────────────────────────────────────────────
# 3. Feature Engineering
# ──────────────────────────────────────────────────────

def add_velocity_features(seq: np.ndarray) -> np.ndarray:
    """
    Augment (T, D) sequence with first-order velocity features.
    Returns (T, 2D) array.
    """
    vel = np.zeros_like(seq)
    vel[1:] = seq[1:] - seq[:-1]
    return np.concatenate([seq, vel], axis=1)


def add_acceleration_features(seq: np.ndarray) -> np.ndarray:
    """
    Augment (T, D) sequence with velocity + acceleration features.
    Returns (T, 3D) array.
    """
    vel = np.zeros_like(seq)
    vel[1:] = seq[1:] - seq[:-1]
    acc = np.zeros_like(vel)
    acc[1:] = vel[1:] - vel[:-1]
    return np.concatenate([seq, vel, acc], axis=1)


# ──────────────────────────────────────────────────────
# 4. Training & Evaluation
# ──────────────────────────────────────────────────────

class BiLSTMTrainer:
    """Encapsulates training and evaluation logic."""
    
    def __init__(self,
                 model: BiLSTMGestureClassifier,
                 lr: float = 1e-3,
                 weight_decay: float = 1e-4,
                 optimizer_name: str = "adamw",
                 class_weights: Optional[torch.Tensor] = None,
                 device: str = "cpu"):
        self.model   = model.to(device)
        self.device  = device
        
        if optimizer_name == "adam":
            self.optimizer = optim.Adam(model.parameters(), lr=lr)
        elif optimizer_name == "adamw":
            self.optimizer = optim.AdamW(model.parameters(), lr=lr,
                                          weight_decay=weight_decay)
        else:
            self.optimizer = optim.SGD(model.parameters(), lr=lr, momentum=0.9)
        
        self.criterion = nn.CrossEntropyLoss(weight=class_weights)
        self.scheduler = optim.lr_scheduler.ReduceLROnPlateau(
            self.optimizer, mode="max", factor=0.5, patience=5
        )
    
    def train_epoch(self, loader: DataLoader) -> float:
        self.model.train()
        total_loss = 0.0
        for seqs, labels, lengths in loader:
            seqs, labels, lengths = (seqs.to(self.device),
                                      labels.to(self.device),
                                      lengths.to(self.device))
            self.optimizer.zero_grad()
            logits = self.model(seqs, lengths)
            loss   = self.criterion(logits, labels)
            loss.backward()
            nn.utils.clip_grad_norm_(self.model.parameters(), max_norm=1.0)
            self.optimizer.step()
            total_loss += loss.item()
        return total_loss / len(loader)
    
    @torch.no_grad()
    def evaluate(self, loader: DataLoader) -> tuple[float, np.ndarray, np.ndarray]:
        self.model.eval()
        all_preds, all_labels = [], []
        for seqs, labels, lengths in loader:
            seqs, lengths = seqs.to(self.device), lengths.to(self.device)
            logits = self.model(seqs, lengths)
            preds  = logits.argmax(dim=-1).cpu().numpy()
            all_preds.extend(preds)
            all_labels.extend(labels.numpy())
        all_preds  = np.array(all_preds)
        all_labels = np.array(all_labels)
        acc = accuracy_score(all_labels, all_preds)
        return acc, all_preds, all_labels
    
    def fit(self,
            train_loader: DataLoader,
            val_loader: DataLoader,
            epochs: int = 50,
            early_stop_patience: int = 10,
            verbose: bool = True) -> dict:
        
        best_val_acc  = 0.0
        patience_cnt  = 0
        history       = {"train_loss": [], "val_acc": []}
        
        for epoch in range(1, epochs + 1):
            loss     = self.train_epoch(train_loader)
            val_acc, _, _ = self.evaluate(val_loader)
            self.scheduler.step(val_acc)
            
            history["train_loss"].append(loss)
            history["val_acc"].append(val_acc)
            
            if val_acc > best_val_acc:
                best_val_acc = val_acc
                patience_cnt = 0
                torch.save(self.model.state_dict(), "/tmp/best_bilstm.pt")
            else:
                patience_cnt += 1
            
            if verbose and epoch % 5 == 0:
                print(f"Epoch {epoch:03d} | loss={loss:.4f} | val_acc={val_acc:.4f}")
            
            if patience_cnt >= early_stop_patience:
                if verbose:
                    print(f"Early stopping at epoch {epoch}")
                break
        
        # Restore best weights
        self.model.load_state_dict(torch.load("/tmp/best_bilstm.pt"))
        return history


# ──────────────────────────────────────────────────────
# 5. Usage Example
# ──────────────────────────────────────────────────────

if __name__ == "__main__":
    from sklearn.model_selection import StratifiedKFold
    
    np.random.seed(42)
    torch.manual_seed(42)
    
    N_CLASSES, N_PER_CLASS = 14, 5
    N = N_CLASSES * N_PER_CLASS
    
    # Simulate variable-length 3D skeleton gestures (21 joints × 3)
    raw_seqs = [np.random.randn(np.random.randint(40, 80), 63).astype(np.float32)
                for _ in range(N)]
    labels = np.repeat(np.arange(N_CLASSES), N_PER_CLASS)
    
    # Feature augmentation
    aug_seqs = [add_velocity_features(s) for s in raw_seqs]  # → (T, 126)
    INPUT_DIM = aug_seqs[0].shape[1]
    
    skf  = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    accs = []
    
    for fold, (train_idx, test_idx) in enumerate(skf.split(aug_seqs, labels)):
        train_seqs = [aug_seqs[i] for i in train_idx]
        test_seqs  = [aug_seqs[i] for i in test_idx]
        
        train_ds = GestureDataset(train_seqs, labels[train_idx])
        test_ds  = GestureDataset(test_seqs,  labels[test_idx])
        
        train_loader = DataLoader(train_ds, batch_size=16, shuffle=True,
                                   collate_fn=collate_fn)
        test_loader  = DataLoader(test_ds,  batch_size=16, shuffle=False,
                                   collate_fn=collate_fn)
        
        model = BiLSTMGestureClassifier(
            input_dim=INPUT_DIM,
            hidden_size=128,
            num_layers=2,
            num_classes=N_CLASSES,
            dropout=0.3,
            pooling="mean",
        )
        
        trainer = BiLSTMTrainer(model, lr=1e-3, optimizer_name="adamw")
        trainer.fit(train_loader, test_loader, epochs=30, verbose=False)
        
        acc, _, _ = trainer.evaluate(test_loader)
        accs.append(acc)
        print(f"Fold {fold+1}: {acc:.4f}")
    
    print(f"\nMean: {np.mean(accs):.4f} ± {np.std(accs):.4f}")
```

---

## 7. Parameters & Ablation Guide

| Parameter        | Candidates                          | Default   | Effect                                                             |     |
| ---------------- | ----------------------------------- | --------- | ------------------------------------------------------------------ | --- |
| `hidden_size`    | 64, **128**, 256                    | 128       | Capacity of each directional LSTM; 2H is output size               |     |
| `num_layers`     | 1, **2**, 3                         | 2         | Depth of the BiLSTM stack; more layers ≠ always better             |     |
| `dropout`        | 0.0, 0.2, **0.3**, 0.5              | 0.3       | Regularization between layers; critical with small datasets        |     |
| `lr`             | 1e-2, **1e-3**, 5e-4, 1e-4          | 1e-3      | Learning rate; use ReduceLROnPlateau to adapt                      |     |
| `optimizer_name` | `adam`, **`adamw`**, `sgd`          | `adamw`   | AdamW adds decoupled weight decay — generally preferred            |     |
| `pooling`        | `last`, **`mean`**, `attention`     | `mean`    | Aggregation strategy over timesteps                                |     |
| `batch_size`     | 8, **16**, 32, 64                   | 16        | Larger batches → less noisy gradients; constrained by dataset size |     |
| `input_features` | raw, **+velocity**, +velocity+accel | +velocity | Feature engineering; velocity is usually free gains                |     |
| `use_input_proj` | **False**, True                     | False     | Linear bottleneck before LSTM; helps if input_dim is very high     |     |
| `weight_decay`   | 0, **1e-4**, 1e-3                   | 1e-4      | L2 regularization for AdamW; prevents overfitting                  |     |

### Parameter Explanations

**`hidden_size`**
Controls the capacity of each LSTM direction. The concatenated BiLSTM output has dimension $2H$. With small gesture datasets (< 500 samples), prefer $H=64$ or $H=128$ to avoid overfitting. With larger datasets, $H=256$ may improve accuracy.

**`num_layers`**
Stacked BiLSTM layers create a temporal hierarchy: lower layers capture local dynamics, higher layers capture gestural phase structure. 2 layers is usually optimal. 3+ layers add marginal benefit and require more data and stronger regularization.

**`dropout`**
Applied between LSTM layers (inter-layer dropout). PyTorch's `nn.LSTM` applies it between layers only — not on the output of the last layer. A post-LSTM dropout is applied explicitly before the classifier head. With very small training sets, use dropout ≥ 0.3.

**`pooling`**
- `mean`: Robust — equally weights all timesteps. Best default.
- `last`: Uses the BiLSTM's "natural summary" (forward endpoint + backward endpoint). Works well when gesture boundaries are clean.
- `attention`: Learns which timesteps matter most. Adds parameters; beneficial when discriminative information is concentrated in specific phases.

**`input_features`**
Raw joint positions are not invariant to subject placement. Adding velocity ($\Delta p_t$) captures movement patterns regardless of starting position. Adding acceleration ($\Delta^2 p_t$) captures dynamic peaks (e.g., flicking motions). Normalize each feature type independently.

**`lr` + `scheduler`**
ReduceLROnPlateau monitors validation accuracy and halves the learning rate when performance plateaus. This often produces the last 2–3% of accuracy gain. Pair with early stopping to avoid overfitting after the last LR reduction.

---

## 8. Complexity

| | Value |
|---|---|
| Parameters (H=128, L=2, D=63, C=14) | ~400K |
| Training time (CPU, 50 epochs, N=70) | ~1–5 min |
| Inference (single gesture, CPU) | <5 ms |
| Memory | O(B × T × H) |

---

## 9. Limitations

- **Data hunger:** Deep learning methods require substantially more data than template-based methods to reach their potential. With fewer than ~200 samples, DTW or $3 may outperform BiLSTM without heavy augmentation.
- **Hyperparameter sensitivity:** BiLSTM performance depends strongly on the combination of `hidden_size`, `dropout`, and `lr`. Grid search is recommended.
- **No user-independent invariance built in:** Unlike DTW and $3, BiLSTM does not have explicit geometric normalization. The model must learn invariances from data, which requires sufficient inter-subject variability in training.
- **Offline by default:** Standard BiLSTM requires the complete gesture before classification. For real-time recognition, a forward-only LSTM or attention-based streaming model is needed.

---

## 10. References

> See `gesture_recognition.bib` for importable BibTeX entries.

- Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory. *Neural Computation*, 9(8), 1735–1780. → [[Papers/Hochreiter1997|Hochreiter1997]]
- Gers, F. A., Schmidhuber, J., & Cummins, F. (2000). Learning to forget: Continual prediction with LSTM. *Neural Computation*, 12(10). → [[Papers/Gers2000|Gers2000]] *(forget gate)*
- Schuster, M., & Paliwal, K. K. (1997). Bidirectional recurrent neural networks. *IEEE Transactions on Signal Processing*, 45(11). → [[Papers/Schuster1997|Schuster1997]]
- Graves, A., Fernández, S., & Schmidhuber, J. (2005). Bidirectional LSTM networks for improved phoneme classification. *ICANN 2005*. → [[Papers/Graves2005_BiLSTM|Graves2005_BiLSTM]]
- Zhang, J. et al. (2021). 3D skeletal joints-based hand gesture spotting and classification. *Applied Sciences*, 11(10), 4689. → [[Papers/Zhang2021_SkeletonBiLSTM|Zhang2021_SkeletonBiLSTM]]
- Zhu, G. et al. (2019). Making skeleton-based action recognition model smaller, faster and better. *ACM MM 2019*. → [[Papers/Zhu2019|Zhu2019]]
- Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. MIT Press. → [[Papers/Goodfellow2016|Goodfellow2016]] *(required citation per guidelines)*
