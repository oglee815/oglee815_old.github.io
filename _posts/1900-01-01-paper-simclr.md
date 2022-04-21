---
layout: post
date: 2021-02-02 14:00
created_date: 2021-02-02 14:00
title: "[Paper] 2002 A Simple Framework for Contrastive Learning of Visual Representations"
author: Ting Chen, Simon Kornblith, Mohammad Norouzi, Geoffrey Hinton
description: A Simple Framework for Contrastive Learning of Visual Representations
comments: true
math: true
category: 
- Paper
tags:
- NLP
- SimCLR
- Contrastive Learning
---

This paper presents SimCLR: a simple framework for contrastive learning of visual representations.
 <!--more-->

## Abstract
> This paper presents SimCLR: a simple framework for contrastive learning of visual representations. We simplify recently proposed contrastive selfsupervised learning algorithms without requiring specialized architectures or a memory bank. In order to understand what enables the contrastive prediction tasks to learn useful representations, we systematically study the major components of our framework. We show that (1) composition of data augmentations plays a critical role in defining effective predictive tasks, (2) introducing a learnable nonlinear transformation between the representation and the contrastive loss substantially improves the quality of the learned representations, and (3) contrastive learning benefits from larger batch sizes and more training steps compared to supervised learning. By combining these findings, we are able to considerably outperform previous methods for self-supervised and semi-supervised learning on ImageNet. A linear classifier traine  on self-supervised representations learned by Sim- CLR achieves 76.5% top-1 accuracy, which is a 7% relative improvement over previous state-ofthe- art, matching the performance of a supervised
ResNet-50. When fine-tuned on only 1% of the labels, we achieve 85.8% top-5 accuracy, outperforming AlexNet with 100x fewer labels.

##  Introduction
- Learning effective visual representations **without human supervision** is a long-standing problem.
- Most mainstream approaches fall into one of two classes: 
  - generative : Generative approaches **learn to generate** or otherwise **model pixels** in the input space.
    - pixel-level generation is computationally expensive and may not be necessary for representation learning.
  - discriminative : Discriminative approaches learn representations **using objective functions similar to those used for supervised learning**, but train networks to perform **pretext tasks** where both the inputs and labels are derived from an unlabeled dataset.
  - Discriminative approaches based on **contrastive learning** in the latent space have recently shown great promise, **achieving state-of-theart results**
- In this work, we introduce a simple framework for contrastive learning of visual representations, which we call **SimCLR**.
- In order to understand what enables good contrastive representation learning, we systematically study the major components of our framework and show that:
  - Composition of multiple **data augmentation operations** is crucial in defining the contrastive prediction tasks that yield effective representations. In addition, unsupervised contrastive learning benefits from **stronger data augmentation** than supervised learning.
  - Introducing a learnable **nonlinear transformation** between the representation and the contrastive loss substantially improves the quality of the learned representations.
  - Representation learning with contrastive cross entropy loss benefits from **normalized embeddings and an appropriately adjusted temperature parameter.**
  - Contrastive learning benefits from **larger batch sizes** and **longer training** compared to its supervised counterpart. Like supervised learning, contrastive learning benefits from deeper and wider networks.
    - > 이 부분 때문에 큰 Batch를 사용하지 않고도 Constrasive learning을 잘 할 수 있는 후속 논문들이 나오는듯 
- We combine these findings to achieve a new **state-of-the-art** in self-supervised and semi-supervised learning on **ImageNet ILSVRC-2012**

##  Method
### The Contrastive Learning Framework
- SimCLR learns representations by **maximizing agreement** between differently augmented views of the same data example via a contrastive loss in the latent space.
- this framework comprises the following four major components.
  - (1)A **stochastic data augmentation module** that transforms any given data example randomly resulting in **two correlated views** of the same example, denoted \\(\tilde{x_{i}}\\) and \\(\tilde{x_{j}}\\) , which we consider as a positive pair. 
    - **random cropping** followed by resize back to the original size, **random color distortions**, and **random Gaussian blur**.
    - the **combination of random crop and color distortion** is crucial to achieve a good performance.
  - (2)A neural network base encoder \\(f(*)\\) that **extracts representation vectors** from augmented data examples. We opt for simplicity and adopt the **commonly used ResNet** to optain \\(h_i = f(\\tilde{x_i}) = ResNet( \\tilde{x_i})\\)
  - (3)A small neural network projection head \\(g(*)\\) that **maps representations to the space where contrastive loss is applied**. 
    - We use a **MLP with one hidden layer** to obtain \\(z_i = g(h_i) = W^{(2)} \sigma(W^{(1)}h_i)\\)
  - (4)A contrastive loss function defined for a **contrastive prediction task**. Given a set \\({\\tilde{x_k}}\\) including a positive pair of examples \\(\tilde{x_{i}}\\) and \\(\tilde{x_{j}}\\), the contrastive prediction task aims to **identify \\(\tilde{x_j}\\) in \\(\\{\tilde{x_k}\\}\_{k!=i}\\) for a given \\(\tilde{x_{i}}\\).**
  - ![framework](/assets/img/simclr_framework.png)

- We randomly **sample a minibatch of N examples** and define the **contrastive prediction task on pairs of augmented examples derived from the minibatch**, resulting in **2N data points**. **We do not sample negative examples explicitly**. Instead, given a positive pair, similar to (Chen et al., 2017), we treat
the other 2(N-1) augmented examples within a minibatch as negative examples. Let \\(sim(u, v) = u^Tv/‖u‖‖v‖\\) denote the dot product between **L2 normalized u and v** (i.e. cosine similarity). 
- Then the loss function for a positive pair of examples (i; j) is defined as ![loss](/assets/img/simclr_loss.png),
- 1[k!=i] is an indicator function evaluating to 1 iff k!=i 
- \\(\tau\\) denotes a temperature parameter.
- The final loss is computed across **all positive pairs**, both (i; j) and (j; i), in a mini-batch.
<!-- - L = \\( \frac{1}{ 2\color{#2196f3}{N} } \sum_{k=1}^{N} [l(2k-1, 2k) + l(2k, 2k-1)] \\) -->
- <span class='centered_small'>![batch](/assets/img/simclr_batch.png)</span>
- ![algorithm](/assets/img/simclr_algorithm.png)

### Training with Large Batch Size
- To keep it simple, we do not train the model with a memory bank (Wu et al., 2018; He et al., 2019). Instead, **we vary the training batch size N from 256 to 8192.**
- A batch size of **8192 gives us 16382 negative examples per positive pair** from both augmentation views. 
- Training with large batch size may be unstable when using standard SGD/Momentum with linear learning rate scaling (Goyal et al., 2017). To stabilize the training, we use the **LARS optimizer** (You et al., 2017) for all batch sizes. Cloud TPUs, using 32 to 128 cores depending on the batch size.
- **Global BN.** Standard ResNets은 Batch norm.을 썼으나 Distributed Data Parallelism 에서는 일반적으로 BN mean and variance를 locally per device 단위로 계산함. 그런데 simCLR에서는 positive pairs가 한 장비에서 학습되기 때문에 모델이 improving representation 없이 local information leakage가 발생 할 수 있음. 따라서 학습 중에 모든 장비에서의 BN mean 과 variance를 모아서 계산함.
  - 이 외에도 accross device 단위로 shuffling data 하거나 replacing BN with layer norm 등의 접근법이 있다.

### Evaluation Protocol
- **Dataset and Metrics** : unsupervised pre-training(encoder f 를 학습하기 위한)은 주로 ImageNet ILSVRC-2012 사용. CIFAR-10에 대한 건 appendix 참조.
  - Pretrained Result를 테스트 하기 위해서는 많이 사용되는 linear evaluation protocol 사용
- **Default setting** : for data augmentation we use random crop and resize (with random flip), color distortions, and Gaussian blur
  - ResNet-50 를 bae encoder로 사용, 2-layer MLP를 사용해서 128-dim latent sapce 으로 projection 함 
  - As the loss, we use **NT-Xent**, optimized using **LARS** with learning
rate of 4.8 (= 0:3 X BatchSize/256) and weight decay of 10e-6.
- Batch는 4096, 100epoch, linear warmup for first 10 epochs


## Data Augmentation for Constrastive Representation Learning

- **Data augmentation defines predictive tasks.** 
  - Architecture 변경이 필요했던 예전 접근과는 달리, 우리는 random cropping with resizing으로 간단히 task와 다른 요소들을 decoupling해서 간단히 만듬.
  - ![crop](/assets/img/simclr_crop.png)

### Composition of data augmentation operations is crucial for learning good representation
- Data augmenation 에 대한 분석을 위해 **두 가지 Type**의 기법 사용
  1. spatial/geometric transformation : cropping, resizing, rotation, cutout
  2. appearance transformation : such as color distortion(color dropping, brightness, contrast, saturation, hue), Gaussian blur, Sobel filtering.
  - ![da](/assets/img/simclr_data_aug.png)
- 각각의 DA 기법에 따른 영향을 보기 위해, individually or pair로 실험 진행
- 항상 randomly crop and resize them to the same resolution은 진행하고,
- Figure2 의 frame 에서 한쪽만 다른 transformation을 진행. 이러면 성능에 영향을 끼칠 수도 있지만 substantively 하지 않다고 판단?
- Figure 5를 보면, single DA로는 positive sample을 잘 찾아내도 충분히 좋은 representation을 생산해낼 수 없는 것으로 보임.
- DA를 Composing 하는 것이 Constrative task는 어렵게 만들지만 represenation은 잘 함.
- **Random crop + RAndom color distortion** 이 제일 결과가 좋았음. 왜냐면 Random crop 만 하면 crop한 이미지들간에는 굉장히 비슷한 **color distribution**을 갖기 때문에 Color만 가지고도 Task를 해결 할 수도 있기 때문
### Contrastive learning needs stronger data augmentation than supervised learning
- 강한 color augmentaion은 linear evaluation of the learned unsuperviese 결과를 향상시켰음(그러나 Supervised learning에서는 benefit이 없었음)

## Architectures for Encoder and Head
### Unsupervised contrastive learning benefits(more) from bigger models
- Depth and width 가 커질수록 당연히 성능이 높아짐.
- 그러나 unsupervised learning이 supervised model보다 큰 크기의 모델에서 오는 장점이 더 큼

### A nonlinear projection head improves the representation quality of the layer before it
- projection head = g(h)
- Projection head 종류 (1) identity mapping; (2) linear projectionand (3) the default nonlinear projection with one additional hidden layer (and ReLU activation)
  - (3)이 제일 성능이 좋았으나, 이 projection head에 들어가기 전 **h가 z=g(h) 보다 성능이 더 좋았다.**
  - 이러한 현상의 원인으로, nonlinear projection 시 information loss가 발생
  - z=g(h)는 data transformation에 invariant하게 훈련이 되므로 
  - 그러므로 g는 downstream task 에 도움이 되는 정보를 제거해 버릴 수도 있다.
  - By leveraging the nonlinear transformation g(), more information can be formed and maintained in h
  - 이를 증명하기 위해 g(h)와 h로 to learn to predict the transformation applied during the pretraining. 을 진행. g(h)는 2048 dim.
  - Table3을 보면 h가 g(h)보다 훨씬 많은 정보를 가지고 있다고 판단됨.
  - ![rep](/assets/img/simclr_rep.png)
  - > 결국 downstream task에는 h가 훨씬 많은 정보를 가지고 있으므로 더 성능이 좋음. 

## Loss Functions and Batch Size
### Normalized cross entropy loss with adjustable temperature works bertter than alternatives
- NT-Xent Loss와 다른 loss들, logistic loss, margin loss와 비교함.
- ![losses](/assets/img/simclr_losses.png)
- L2 Norm 과 적절한 Temperature scaling 이 없으면 성능이 굉장히 떨어짐.
- ![l2](/assets/img/simclr_l2_temp.png)

### Constrastive learning benefits (more) from larger batch sizes and longer training
- Epoch이 작을 때는(e.g. 100) larger batch size가 굉장히 도움이 되고, epoch이 커지면 이 효과가 감소하거나 없어짐
- Supervised 와 다르게, 큰 배치가 더 많은 negative examples 를 가져오고 facilitating convergence를 한다. 

## Comparison with State-of-the-art
- **Linear evaluation** : 
- ![eval1](/assets/img/simclr_linear_eval.png)
- **semi-supervised learning**: 
- > few-labels training을 semi-superv라고 한듯?
- ![eval2](/assets/img/simclr_semi.png)
- **Transfer learning**
- ![eval3](/assets/img/simclr_transfer.png)


## Conclusion
- In this work, we present a simple framework and its instantiation for contrastive visual representation learning.
- Our approach differs from standard supervised learning on ImageNet only in the choice of **data augmentation, the use of a nonlinear head at the end of the network, and the loss function.**