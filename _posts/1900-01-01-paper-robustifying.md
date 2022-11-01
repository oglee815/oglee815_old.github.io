---
layout: post
date: 2022-05-9 14:00
created_date: 2022-05-9 14:00
title: "[Paper] 2107 Robustifying Multi-hop QA through Pseudo-Evidentiality Training"
author: Kyungjae Lee,  Seung-won Hwang, Sang-eun Han, Dohyeon Lee
description: " Robustifying Multi-hop QA through Pseudo-Evidentiality Training"
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Multi-hop QA
---

Robustifying Multi-hop QA through Pseudo-Evidentiality Training
<!--more-->

# Abstract
- This paper studies the **bias problem** of multihop question answering models, of answering correctly **without correct reasoning.** One way to robustify these models is by supervising to not only answer right, but also **with right reasoning chains**. An existing direction is to annotate reasoning chains to train models, requiring expensive additional annotations. In contrast, we propose a new approach to learn **evidentiality**, deciding whether the answer prediction is supported by correct evidences, without such annotations. Instead, we compare counterfactual changes in answer confidence with and without evidence sentences, to generate “pseudo-evidentiality” annotations. We validate our proposed model on an **original set and challenge set in HotpotQA**, showing that our method is accurate and robust in multi-hop reasoning.

# Introduction
- Multi-hop question answering is a task of answering complex questions by connecting information from several texts.
- However, previous works observe "disconnected reasoning" in some correct answer.
- It happens when models can exploit specific types of artifacts (e.g., entity type), to leverage them as **reasoning shortcuts**
  - 질문이 시간을 의미하면 대충 시간이라는 entitiy의 답을 찾는다든가..
- To address the problem of reasoning shortcuts, we propose to supervise **evidentiality** - decideing whether a model answer is supported by correct evidences.
- This is related to the problem that most of the early reader models for QA failed to predict **wether questions are not answerable**.
- Lack of answerability training led models to provide a wrong answer with high confidence, **when they had to answer 'unanswerable'**
- similarly, we aim to train for models to recognize **whether their answer is "unsupported" by evidences**, as well.
- Answerability와 함께, **(1) Evidence-positive, (2) Evidence-negative set**을 만듦
#### Question 1: 그럼 어떻게 저 데이터셋을 만들 것인가?
  - 과거 기법 : Attention score -> pseudo annotation for evidence-positive set. 그러나 명확한 인과관계가 부족하고 multiple evidence를 sum해야하는 부분이 부족
  - **따라서 Interpreter module을 추가하여, evidence들을 그룹핑 하는 작업을 수행**
#### Question 2: 어떻게 학습시킬 것인가?
- Objective1 : QA model should not be overconfident in evidence-negative set
- Objective2 : confident in  evidence-positive
- For O1, lower the model confidence on evidence-negative set via regularization. 
  - However such regularization can cause violating O2 due to correlation between confidence distributions for evicence-positive and negative set(무슨 말인지 모르겠음)
  - Our solution is to **selectively regularize**, by purposedly training a biased model violating (O1), and decorrelate the target model from the biased model.

# Approach
#### Generating Examples for training answerability and evidentiality
##### Answerability
- We build triples of question Q, an answer A, and passage D, to be labeled for answerability.
- (Q, A, D) -> answer-positive A+, unanswerable set -> A-
- From this labels, we train a transformer-based model to classify the answerability.
- 그러나 answerability하다고 해도 evidentiality를 보장해주지는 않음

##### Evidentiality
- 정답은 있으나, 정답을 유추할 수 있는 reason이 없으면 -> Evidence-negative E-, 있으면 Evidence-positive E+
- 근데 레이블이 없으니 pseudo-evidentiality를 사용
- 1) Answer sentence Only : we remove all sentences in answerable passage except S*(sentences containing an answer), such that the input passage D becomes S*, which contains a correct answer but no other evidences. 
  - 이거 근데 그냥 Sentence 1개만 있다는 말 아냐?
- 2) Answer Sentence + Irrelevant Facts : concat S* + unanswerable D
- 3) Partial Evidence + Irrelevant Facts
- Positive Set은 학습된 모델이 정답을 예측하는데 가장 큰 영향을 준 sentence로 정함. 
  - (a) 정답을 포함한 문장과 다른 문장들을 하나씩 추가해보면서 가장 높은 Prob을 내는 Set을 Evidence Set으로 함
  - (b) 문장을 뻇을때 prob이 가장 줄어드는 문장을 evidence sentence로
  - (a) + (b) 로 함
