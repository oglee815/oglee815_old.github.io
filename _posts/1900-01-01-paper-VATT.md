---
layout: post
date: 2022-03-02 14:00
created_date: 2022-03-02 14:00
title: "[Paper] 2112 VATT, Transformers for Multimodal Self-Supervised Learning from Raw Video, Audio and Text"
author: Hassan Akbari et al @ columbia univ and google
description: transformers for Multimodal Self-Supervised Learning from Raw Video, Audio and Text
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Multimodal
- Transformer
---

We present a framework for learning multimodal representations from unlabeled data using convolution-free Transformer architectures
<!--more-->

## Abstract
> We present a framework for learning multimodal representations from unlabeled data using convolution-free Transformer architectures. Specifically, our VideoAudio-Text Transformer (VATT) takes **raw signals as inputs and extracts multimodal representations** that are rich enough to benefit a variety of downstream tasks. 
We train VATT end-to-end from scratch **using multimodal contrastive losses** and evaluate its performance by the downstream tasks of **video action recognition, audio event classification, image classification, and text-to-video retrieval**. 
Furthermore, we study a **modality-agnostic,** single-backbone Transformer by sharing weights among the three modalities. We show that the convolution-free VATT outperforms state-of-the-art ConvNet-based architectures in the downstream tasks. Especially, VATT’s vision Transformer achieves the top-1 accuracy of 82.1% on Kinetics-400, 83.6% on Kinetics-600, 72.7% on Kinetics-700, and 41.1% on Moments in Time, new records while **avoiding supervised pre-training.** Transferring to image classification leads to 78.7% top-1 accuracy on ImageNet compared to 64.7% by training the same Transformer from scratch, showing the generalizability of our model despite the domain gap between videos and images. VATT’s audio Transformer also sets a new record on waveform-based audio event recognition by achieving the mAP of 39.4% on AudioSet without any supervised pre-training. VATT’s source code is publicly available.2

## Intro
- 왜 Transformer 인가?
  - Convolutional neural networks (CNNs) [53, 51] have triumphed over various computer vision tasks.
  - The inductive bias induced by convolutions, namely **translation invariance and locality**, are proven effective for the visual data. 
  - however, we witness in the natural language processing (NLP) community a **paradigm shift from the models with strong inductive biases**, such as recurrent neural networks [43, 7] and CNNs [104, 32], to more general architectures constructed upon self-attention
  - Their work(Transformer models) delivered a compelling message that “large scale (supervised) training trumps inductive bias (for image classification).”
- 그러나 label 데이터가 별로 없음
- Hence, this work poses another pressing question about the Transformers that take raw signals as input. **How to empower them with large-scale, unlabeled visual data?**
- For visual data, the most **organic supervision** is arguably the **multimodal videos**.
- We call the **video, audio, text Transformers VATT**.
- VATT borrows the exact architecture from **BERT** [23] and **ViT** [25] except the layer of tokenization and linear projection reserved for each modality separately
- evaluation tasks: 
  - image classification, video action recognition, audio event classification, and zero-shot text-to-video retrieval.
- To move one step forward, we challenge the Transformers in VATT by a seemingly too strong constraint: **sharing weights among the video, audio, and text modalities.**
  - The idea is to test whether there exists a single, general-purpose model for all the modalities — of course, they still have their own layers of tokenization and linear projection.
- **DropToken**, a simple and yet effective technique to reduce the training complexity
  - DropToken **randomly drops a portion of the video and audio tokens** from each input sequence during training, allowing for high-resolution inputs and leveraging their abundance

## Related work
- Transformers in Vision
- Self-supervised Learning
  - Single vision Modalities
  - Multimodal video
  - **VATT serves as a first work combining the strength of convolution-free Transformer and multimodal contrastive learning.**

## Approach
- <span>![app](/assets/img/vatt/approach.jpg)</span>
- We feed each modality to a tokenization layer, where the raw input is projected to an embedding vector followed by a Transformer
- Two major settings: 
  - 1) The backbone Transformers are **separate and have specific weights for each modality**
  - 1) The Transformers **share weights, namely, there is a single backbone Transformer** applied to any of the modalities. In either setting, the backbone extracts modality-specific representations, which are then mapped to common spaces to be compared with each other by contrastive losses.

### Tokenization and Positional Encoding
- VATT operates on **raw signals**. 
  - The vision-modality input consists of 3-channel RGB pixels of video frames
  - the audio input is in the form of air density amplitudes (waveforms), 
  - the text input is a sequence of words
- We partition an entire video clip of size T × H × W to a sequence of [T / t] · [H / h] · [W / w] patches, where each patch contains t × h × w × 3 voxels.
  > A voxel is a unit of graphic information that defines a point in three-dimensional space. Since a pixel (picture element) defines a point in two dimensional space with its X and Y coordinates , a third z coordinate is needed. In 3-D space, each of the coordinates is defined in terms of its position, color, and density. Think of a cube where any point on an outer side is expressed with an x , y coordinate and the third, z coordinate defines a location into the cube from that side, its density, and its color. With this information and 3-D rendering software, a two-dimensional view from various angles of an image can be obtained and viewed at your computer.
- This can be seen as a **3D extension** of the patching mechanism proposed in **ViT.**
  - <span>![pos](/assets/img/vatt/positional.JPG)</span>
- The raw audio waveform is a 1D input with length T' , and we partition it to [T' / t'] segments each containing t' waveform amplitudes.
- For text, we first construct a vocabulary of size v out of all words in our training dataset. 
- For an input text sequence, we then map each word to a v-dimensional one-hot vector followed by a linear projection with a learnable weight matrix
- This is equivalent to an embedding dictionary lookup, which has been widely used in natural language understanding(**Word2Vec**)

### DropToken
- we randomly sample a portion of the tokens and then feed the sampled sequence, **not the complete set of tokens**, to the Transformer.
- We argue that instead of reducing the resolution or dimension of the raw inputs, it is better to keep a high-fidelity(정확한) input and randomly sample the tokens via DropToken.

### Transformer architecture
- Similar to ViT [25], we do not tweak the architecture so that our weights can be easily transferred to any standard Transformer implementation. 

### Common Space Projection
- We use common sapce projection and contrastive learning in that common space to train our networks
- More specifically,  given a video-audio-text triplet, we define **a semantically hiderarchical common space mapping** that enabels us to directly compare **video-audio** paires as well as **video-text** pairs by the cosine similarity.
- To achieve this, we define multi-level projections as follows:
  - <span>![prj](/assets/img/vatt/projection.JPG)</span>
- The main intuition behind this hierarchy is that different modalities have **different levels of semantic granularity**, so we should impose this as an inductive bias in the common sapce projection

### Multimodal contrastive Learning
- We use **Noise Contrastive Estimation(NCE)** to align video-audio paris and **Multiple Instance Learning NCE(MIL-NCE)** to align video-text-pairs.
- Positive paris from two modalities are constructed by sampling their corresponding streams from the **same location** in the video
- Negative pairs are constructed by sampling from any **non-matching locations** in the video.
- <span>![loss](/assets/img/vatt/loss.JPG)</span>
- **N** contains all non-matching paris in a batch
- **P** contains five text clips that are nearest neighbors to the video colp in time.
- The overall per-sample objective for training the entire VATT model end-to-end is as follows:
  - <span>![loss2](/assets/img/vatt/loss2.JPG)</span>
- <a class="color_a" href="https://nuguziii.github.io/survey/S-006/">NCE Loss</a>
- **Noise Contrastive Estimation? (위 링크 펌)**
Instance discrimination을 하기 때문에 저희는 N개의 이미지에 대해 N-classification을 합니다. 하지만, 이미지의 개수가 1억장이면 1억개의 classification을 하는 것은 불가능 하기 때문에, 우리는 approximate 합니다. positive, negative 인 binary classification으로 생각을 하고, 임의의 sample 안에서 positive와 negative 클래스만 구별을 해 전체를 approximate 합니다.
- <span>![nce](/assets/img/vatt/infoNCE.png)</span>
