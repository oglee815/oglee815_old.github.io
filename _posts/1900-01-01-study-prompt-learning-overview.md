---
layout: post
date: 2022-05-10 14:00
created_date: 2022-05-10 14:00
title: "[Study] Prompt Learning overview"
author: Oglee
description: Prompt-Based Learning
comments: true
math: true
category: 
- Study
tags:
- NLP
- Prompt learning
---

Prompt Learning overview(대부분의 출처 : https://www.youtube.com/watch?v=q5FGZBqK-vc)
<!--more-->

# Introduction
- GPT3의 Parameter는 **175B**이다. 과연 우리가 이걸 Fine-tuning 할 수 있을까?
  - '할 수 있다 없다'의 문제가 아니라 **할 필요가 없다.**
  - Zero-shot, Few-shot Learning이 가능하기 때문.
  - ![그림1](https://user-images.githubusercontent.com/18374514/167583955-f0d91cfc-6185-43eb-ae1c-2e46e622ad69.png)[1]
  - 이게 가능한 이유는 모델의 **pretraining 방식이 In-context learning과 동일하기 때문**
    - ![image](https://user-images.githubusercontent.com/18374514/167589381-8be3d23c-97bc-4121-a51a-d0a2f21c2a53.png)
    - 위 그림에서 처럼, Generative 방식의 Pretraining을 하다보면, 저절로 Translation Task를 학습하는 것과 비슷한 학습을 하게 된다.
    - 즉, Context를 배우는 것 자체가 Task를 배우는게 되버리는 현상이 생긴다. 이것이 In-Context Learning
    - ![image](https://user-images.githubusercontent.com/18374514/167590581-a71a1f3f-304f-47a0-b22f-3cecb317e027.png)
  - GPT3의 경우, 일부 Task에서는 Few-shot 성능이 Fine-tune 성능을 넘어서고 있다.
    - ![image](https://user-images.githubusercontent.com/18374514/167585189-9d5503eb-154b-4ba9-9813-910749056fc0.png)
  - 그럼 다음에 떠오르는 질문은,
    - Few-shot 혹은 Zero-shot 성능을 높이는 방법이 뭔데? 
    - 그리고 우리는 GPT-3같은 모델 없는데 어떡함?ㅠ

# Few-shot 혹은 Zero-shot 성능을 높이는 방법이 뭔데? 
- Prompt Design(Engineering)을 잘하면 된다.
- Prompt란? Language 모델이 문제를 Few-shot, Zero-shot으로도 잘 풀 수 있도록 입력을 잘 변형하는 것.
  - ![image](https://user-images.githubusercontent.com/18374514/167590914-152aa992-2d13-4b6d-8c10-8cd82b406a45.png)[1]
- 그런데 이 Prompt Engineering을 hand crafted 한 방식으로 정하게 되면 문제가 생긴다.
  - ![image](https://user-images.githubusercontent.com/18374514/167592030-360ae6aa-552f-453f-a38e-26ebb0b0d40e.png)
- **GPT understands, Too(Liu, et al. 2021)** 와 같은 논문에서 자동으로 Prompt를 Search 할 수 있는 방식 제안
  - [2]![image](https://user-images.githubusercontent.com/18374514/167594922-af313388-66e8-49af-9d8a-e60ea97427a4.png)
    - 위 그림의 왼쪽과 같은 경우, 최적의 단어 조합을 찾는 것 자체가 Discrete 한 space에서 최적화를 진행하는 방식이다. 이는 Neural Net의 최적화에는 Sub-optimal 할 수 있다.
    - 따라서 본 논문에서는 오른쪽 방식과 같이, 몇 개의 중요한 anchor token을 제외하고, 나머지는 Bi-LSTM으로 이루어진 Encoder의 output인 **continuous vector를 prompt로 주고 이를 최적화 하는 continuous search 방식을 제안**한다.
  - ![image](https://user-images.githubusercontent.com/18374514/167595838-28f6b1e5-f644-4173-ab56-7999bf7ff0be.png)[2]
    - 그 결과, NLU Task에서는 BERT를 이길 수 없었던 과거를 깨고 **SuperGLUE에서도 GPT가 BERT를 능가하게 됨.**
- 가장 중요한 점은, **Prompt Learning에서는 GPT나 BERT와 같은 PLM은 Fixed 시키고 Prompt Encoder 만 학습 시킨다는 점**

# 우리는 GPT-3 같은 모델 없는데 어떡함?ㅠ
- BERT와 같은 작은 모델에서도 NLU Task에 Prompt를 적용하는 연구 등장
  - P-Tuning v2: Prompt Tuning Can Be Comparable to Fine-tuning Universally Across Scales and Tasks
  - DCPCSE : Deep Continuous Prompt for Contrastive Learning of Sentence Embeddings
- DCPCSE의 예시
  - ![image](https://user-images.githubusercontent.com/18374514/167597707-619bb5bb-96fe-4e17-a5bd-fecba1615e52.png)
  - 기존 방식(like SimCSE)에서는 위 그림의 파란색 부분(BERT와 같은 PLM)을 열심히 Fine-tuning해서 Sentence Embedding을 뽑아냈다면,
  - DCPCSE에서는 위 그림의 빨간색 부분만 학습해서 Sentence Embedding을 수행함
  - 빨간색 부분의 파라미터 갯수는 많아야 파란색 대비 2%정도

# Reference
- [1]https://www.youtube.com/watch?v=q5FGZBqK-vc
- [2]https://arxiv.org/abs/2103.10385
