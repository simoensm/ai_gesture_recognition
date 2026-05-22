---
title: "ImageNet Classification with Deep Convolutional Neural Networks (AlexNet)"
authors: ["Krizhevsky, Alex", "Sutskever, Ilya", "Hinton, Geoffrey E."]
year: 2012
citekey: "Krizhevsky2012_AlexNet"
doi: "10.1145/3065386"
venue: "NeurIPS 2012 / Communications of the ACM, 60(6)"
type: conference
tags: [literature, cnn, deep-learning, imagenet, relu, dropout, gpu-training, foundational]
status: read
relevance: foundational
created: 2026-05-22
---

# ImageNet Classification with Deep Convolutional Neural Networks (AlexNet)

> [!info] Metadata
> **Authors:** Alex Krizhevsky, Ilya Sutskever, Geoffrey E. Hinton
> **Year:** 2012
> **Venue:** NeurIPS 2012

---

## Abstract

AlexNet won the 2012 ImageNet Large Scale Visual Recognition Challenge (ILSVRC) with a top-5 error of 15.3%, compared to 26.2% for the second-place entry. This 10+ percentage point gap is widely considered the moment that launched the deep learning revolution.

---

## Key Contributions

- **ReLU activations:** Much faster training than sigmoid/tanh; avoids vanishing gradients
- **GPU training:** Trained on 2 GTX 580 GPUs — showed GPUs are essential for large-scale DL
- **Dropout (0.5):** First large-scale use of dropout as regularisation
- **Data augmentation:** Crops, flips, colour jitter — standard practice since
- **Local Response Normalisation (LRN):** Now replaced by batch norm
- **Overlapping max pooling**
- 8 layers (5 conv + 3 FC), 60M parameters

## Results

- ILSVRC 2012: 15.3% top-5 error (second place: 26.2%)
- Triggered massive investment in deep learning and GPU computing

## Connections

- Architecture → [[Concepts/Convolutional_Neural_Networks|CNN]]
- Dropout → [[Concepts/Bias_Variance_Regularisation|Regularisation]]
- Improved by → [[Papers/He2016_ResNet|ResNet]]
