---
title: "Convolutional Neural Networks (CNNs)"
tags: [concept, cnn, deep-learning, computer-vision, convolution, pooling]
created: 2026-05-22
---

# Convolutional Neural Networks (CNNs)

> [!abstract] The inductive bias of locality
> CNNs exploit two structural priors: (1) nearby pixels/features are correlated, (2) useful patterns can appear anywhere (translation equivariance). These assumptions dramatically reduce parameters and improve generalisation on spatial data.

**Key papers:** [[Papers/LeCun1989_LeNet|LeCun 1989 (LeNet)]], [[Papers/Krizhevsky2012_AlexNet|Krizhevsky 2012 (AlexNet)]], [[Papers/He2016_ResNet|He 2016 (ResNet)]]

---

## 1. The Convolution Operation

A 2D convolution of input $X$ with kernel $K$:
$$(X * K)_{i,j} = \sum_{m}\sum_{n} X_{i+m,\, j+n} \cdot K_{m,n}$$

**Key properties:**
- **Weight sharing:** the same kernel applied at every position → massive parameter reduction
- **Translation equivariance:** if input shifts, feature map shifts by the same amount
- **Local receptive field:** each output depends only on a small spatial region of the input

---

## 2. Architecture Components

### Convolution Layer
- Input: $(H \times W \times C_{\text{in}})$
- Kernel: $(k \times k \times C_{\text{in}} \times C_{\text{out}})$
- Output: $(H' \times W' \times C_{\text{out}})$
- Parameters: $k^2 \cdot C_{\text{in}} \cdot C_{\text{out}} + C_{\text{out}}$ (bias)

### Pooling Layer
- **Max pooling:** takes the maximum in each $k \times k$ window
- **Average pooling:** takes the mean
- **Global average pooling:** reduces each feature map to a single value → used before the classifier

### Fully Connected (Dense) Layers
- After spatial feature extraction, flatten → MLP → class probabilities

### Batch Normalisation
- After convolution, before activation
- Accelerates training, acts as regulariser

---

## 3. Architectural Evolution

| Architecture | Year | Innovation | Top-1 on ImageNet |
|---|---|---|---|
| **LeNet-5** | 1989/1998 | First successful CNN (digit recognition) | — |
| **AlexNet** | 2012 | ReLU, dropout, GPU training, 8 layers | 63.3% |
| **VGGNet** | 2014 | Very deep with 3×3 kernels (16–19 layers) | 74.4% |
| **GoogLeNet/Inception** | 2014 | Inception modules, 22 layers | 74.8% |
| **ResNet** | 2016 | Residual connections → 152 layers | 80.7% |
| **DenseNet** | 2017 | Dense connections between all layers | 82.8% |
| **EfficientNet** | 2019 | Compound scaling of depth/width/resolution | 84.3% |
| **ViT** | 2020 | Patches as tokens → transformer applied to images | >87% |

---

## 4. Residual Connections (ResNet)

The key innovation that enabled very deep networks:
$$y = F(x, W) + x$$

The identity shortcut $x$ allows gradients to flow directly backward, bypassing the non-linear layers. This avoids vanishing gradients at depth.

**Interpretation:** The network learns the *residual* $F(x) = H(x) - x$ (what to add to the input) rather than the full mapping $H(x)$.

---

## 5. Receptive Field

After $L$ layers with kernel size $k$: receptive field grows as $\mathcal{O}(k^L)$.

**Dilated convolutions:** expand receptive field without increasing parameters by introducing gaps in the kernel.

---

## 6. Transfer Learning

Pre-trained CNNs (ImageNet weights) can be fine-tuned on small datasets:
1. **Feature extraction:** freeze all layers, train only final classifier
2. **Fine-tuning:** unfreeze later layers, train with small LR

This is highly effective and explains why pre-trained CNNs are used in most practical vision tasks.

---

## Connections

- Backprop through conv layers → [[Backpropagation]]
- Deeper networks → skip connections → [[Papers/He2016_ResNet|ResNet]]
- CNN replaced by attention for images → [[Attention_Transformer]]
- Applied to 1D time series → can replace RNNs for gesture recognition
- Historical breakthrough → [[Papers/Krizhevsky2012_AlexNet|AlexNet]]
