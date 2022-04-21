---
layout: post
date: 2020-09-17 14:08
created_date: 2020-09-17 14:08
title: "[Paper] (작성중) 1904 UNSUPERVISED DATA AUGMENTATION FOR CONSISTENCY TRAINING"
author: Qizhe Xie et.al, Google Research, Brain Team, Carnegie Mellon University
description: UNSUPERVISED DATA AUGMENTATION FOR CONSISTENCY TRAINING
comments: true
math: true
category: 
- Paper
tags:
- Transformers
- Data augmentation
- NLP
---

작성중
 <!--more-->
# Abstract

Semi-supervised learning lately has shown much promise in improving deep learning models when labeled data is scarce. Common among recent approaches is the use of consistency training on a large amount of unlabeled data to constrain model predictions to be invariant to input noise. In this work, we present a new perspective on how to effectively noise unlabeled examples and argue that the quality of noising, speciﬁcally those produced by advanced data augmentation methods, plays a crucial role in semi-supervised learning. By substituting simple noising operations with advanced data augmentation methods such as RandAugment and back-translation, our method brings substantial improvements across six language and three vision tasks under the same consistency training framework. On the IMDb text classiﬁcation dataset, with only 20 labeled examples, our method achieves an error rate of 4.20, outperforming the state-of-the-art model trained on 25,000 labeled examples. On a standard semi-supervised learning benchmark, CIFAR-10, our method outperforms all previous approaches and achieves an error rate of 5.43 with only 250 examples. Our method also combines well with transfer learning, e.g., when ﬁnetuning from BERT, and yields improvementsinhigh-dataregime,suchasImageNet,whetherwhenthereisonly 10% labeled data or when a full labeled set with 1.3M extra unlabeled examples is used.1
