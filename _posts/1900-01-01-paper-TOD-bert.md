---
layout: post
date: 2021-08-31 14:00
created_date: 2021-08-31 14:00
title: "[Paper] 2010 TOD-BERT, Pre-trained Natural Language Understanding for Task-Oriented Dialogue"
author: Chien-Sheng Wu, Steven Hoi, Richard Socher, and Caiming Xiong
description: TOD-BERT paper
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Task Oriented Dialogue
- TOD
---

BERT와 Contrastive objective modeling 방식을 TOD에 적용
 <!--more-->

## Abstract

> The underlying difference of linguistic patterns between general text and task-oriented dialogue makes existing pre-trained language models less useful in practice. In this work, we unify nine human-human and multi-turn task-oriented dialogue datasets for language
modeling. To better model dialogue behavior during pre-training, we incorporate user and system tokens into the masked language
modeling. We propose a contrastive objective function to simulate the response selection task. Our pre-trained task-oriented dialogue BERT (TOD-BERT) outperforms strong baselines like BERT on four downstream taskoriented dialogue applications, including intention recognition, dialogue state tracking, dialogue act prediction, and response selection. We also show that TOD-BERT has a stronger
few-shot ability that can mitigate the data scarcity problem for task-oriented dialogue.

## Introduction

- BERT, RoBERTa 등의 PLM을 활용하여 English Wikipedia or books의 NLU에 활용하는 사례는 많으나, conversational corpora에 바로 이용하기는 어려움.
- One possible reason could be the **intrinsic difference of linguistic patterns** between human conversations and writing text, resulting in a large gap of data distributions
  - Convesational Text에 BERT를 그대로 사용하기 힘든점(From PLATO paper)
    1. the underlying linguistic patterns in human conversations can be highly different from those in general text, which suggests a potentially large gap in knowledge or data distribution
    2. the training mode of uni-directional dialogue generation is also distinct from that of bi-directional natural language understating as applied in BERT 
    3. unlike most of the general NLP tasks, there exists a one-to-many relationship in dialogue generation, where the dialogue context may correspond to multiple appropriate replies.
- 따라서 Twitter or Reddit, social media corpus를 사용해서 Dialogue language model을 훈련하는 경우가 많음.
- 그런 데이터들은 대부분 short, noisy, without specific chating goals.
- **a task-oriented dialogue has explicit goals (e.g. restaurant reservation or ticket booking) and many conversational interactions.**
  - TOD는 유저나 시스템의 행동에 목적이 정해져있기 때문에 NLU의 중요성이 더 부각됨.
- <u>This paper aims to prove this hypothesis: </u>
  - using **task-oriented corpora can learn better representations** than existing pre-trained models for task-oriented downstream tasks
- 9개 데이터셋, 100k dialogues with 1.4M utterances across over 60 difference domains
- BERT를 활용하되, MLM 외에 Response selection을 위한 <span class='my_highlight'>Contrastive objective</span>를 추가함
- Test를 위해서 사용한 Tasks :
  - intention recognition, dialogue state tracking, dialogue act prediction, and response selection
- 결과적으로 TOD-BERT가 BERT, GPT, DialGPT보다 결과가 좋음.
- 특히 Few-shot ability 에서 강점을 보임


##  Related Work
- **General Pre-trained Language Models**: GPT류의 left-to-right model, BERT류의 Bi-directional 모델, UniLM과 같은 uni,bi의 혼합 존재, Conditional LM(CTRL)
- **Dialogue Pre-trained Language Models**: 레딧, 트위터와 같은 open-domain conversational data에 훈련된 모델들. 
  - Transfertransfo, DialoGPT, 
  - ConveRT(pre-trained a dual transformer encoder for response selection task on large-scale Reddit (input, response) pairs, 
  - PLATO uses both Twitter and Reddit data to pre-trained a dialogue generation model with discrete latent variables
- **Pretraining for task-oriented dialogues**:
  - `Hello, It’s GPT-2 - How Can I Help You?`라는 논문에서 gpt2를 reponse generation task에 사용
    - response generation task? which takes system belief, database result, and last dialogue turn as input to predict next system responses
    - ![d](/assets/img/TOD1.png)
  - `Training neural response selection for task-oriented dialogue systems.`는 response selection model에 이용
  - `Few-shot natural language generation for task-oriented dialog`, which assumes dialogue acts and slot-tagging results are given to generate a natural language response

## Method
### Datasets
- <span class='centered_small'>![dataset](/assets/img/TOD2.png)</span>
  - 9 different TOD dataset, english, human-human, multi-turn, 100,707 dialogues, 1,388,152 utterances over 60 domains
- MetaLWOZ: Meta-Learning Wizard-of-Oz, predicting user response, covering 227 tasks in 47 domains.
- Schema: dialogue state tracking dataset
- Taskmaster: Spoken and written dialogues created with two distinct procedures.
- MWOZ: MultiDomain Wizard-of-Oz dataset, It has a detailed description of the data collection procedure, user goal, system act, and dialogue state labels
- MSR-E2E: Microsoft end-toend dialogue challenge, movie-ticket booking, restaurant reservation, and taxi booking
- SMD: Stanford multidomain dialogue is an in-car personal assistant dataset,
- Frames: book a trip and assistants who search a database to find appropriate trips. Unlike other datasets, it has labels to keep track of different semantic frames, which is the decision-making behavior of users throughout each dialogue
- WOZ and CamRest676 

### TOD-BERT
- BERT-base uncased model 사용
- To capture speaker information and the underlying interaction behavior in dialogue, we add two special tokens, <span class='my_highlight'>[USR] and [SYS]</span> 
- We prefix the special token to each user utterance and system response, and concatenate all the utterances in the same dialogue into one flat sequence, with standard positional embeddings and segmentation embeddings.
  > user와 system이 각 한 개씩이기 때문에 스페셜 토큰이 각 하나씩 필요한듯? Segment embedding은 어떻게 적용?
- **Masked Language modeling** : BERT에서와 비슷
- **Response contrastive loss** : 
  - It does not require any additional human annotation.
  - Advantages:
    - [CLS]를 보다 잘 훈련시킬 수 있다.
    - we encourage the model to capture underlying <u>dialogue sequential order, structure information, and response similarity.</u>
  - we apply a **dual-encoder approach** instead of NSP objective, and simulate **multiple negative samples**.
    > Dual Encoder라고 하는 이유는 CLS를 각각 두 개의 Representation으로 만들어서 학습하기 때문
    - ConveRT 그림 (Dual Encoder 구조)
    - <span class='centered_small'>![convrt](/assets/img/TOD4.png)</span>
    - Contrastive 학습 순서
    1. draw a batch of dialogues $$\{D_1, D_2, ..., D_b\}$$
    2. split each dialogue at a randomly selected turn t. For example, <span class='my_highlight'>$$\{D_1\}$$ will be sepreated into two segments</span>, one is the context $$\{S^1_1, U^1_1, ..., S^1_t, U^1_t\}$$ and the other is the response $$\{S^1_{t+1}\}$$
    > 여기서 진짜 중요한 점은 Context와 response를 나눈 다음 각각 따로 encoder 에 넣는다는 점. 그래서 2개의 CLS의 representation을 만들 수 있음.
    3. 그 다음, Context Matrix $$C \in R^{b × d_B}$$와 Response Matrix $$R \in R^{b × d_B}$$에 각각 CLS토큰을 통과시켜서 representation을 뽑아낸다.
    4. 다른 배치의 샘플들은 모두 negative sample로 간주(for contrasive learning)
    5. C와 R을 통과시킨 각각의 결과는 batch X 768이 되고, 이 둘을 곱하면 B X B 매트릭스가 된다.
    6. B X B 매트릭스가 의미하는 것은 각 샘플이 서로 positive(알맞은 response)냐 negative냐를 의미한다. 즉 diagonal 요소들만 positive, 나머지는 다 negative다. B X B 값을 softmax 통과시키면 최대값이 1인 확률이 된다.(알맞은 response를 찾아낼 확률을 높이도록 학습되겠지?)
    7. 하단의 loss 식을 보면 $$M_{i,i}$$ 만 1로 만들려는 식임을 알 수 있다.(i,i 는 대각성분을 의미)
      - ![rcl](/assets/img/TOD3.png)
