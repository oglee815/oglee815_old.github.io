---
layout: post
date: 2022-04-06 00:00
reviewed_date: 2022-04-06 14:00
title: Prompt Learning Paper Concepts Summary
author: oglee
description: Prompt Learning
comments: true
math: true
category:
- Paper
tags:
- NLP
- Prompt Learning
---

Prompt Learning Paper Concept 정리
 <!--more-->

| Title | Author | Date | Summary |
|---|---|---|---|
| Prefix-Tuning : Optimizing Continuous Prompts for Generation | Li and Liang, Stanford | 2101 | - GPT3에 "TL;DR"을 prepend해서 summary를 하는 prompting 혹은 in-context learning에서 영감을 받음 <br>- Encoder와 decoder 입력 앞에 virtual token(prefix)을 붙여서, 해당 토큰과 관련된 파라미터만 학습함(Continuous vector) <br>- prefix 토큰의 임베딩 메트릭스를 곧바로 학습하지 않고 reparameterize를 시켜서 학습함. p = MLP(p'), inference 시에는 p만 사용 <br>- Task : Table-To-Text, Abstractive Sum <br>- 흥미로운 결과: Prefix를 랜덤보다 그냥 banana라도 real word로 하는게 성능이 좋음, prefix vs infix 끼리 비교도 함 <br>- P-tuning하고 유사한 continuous prompt learning이지만 이 페이퍼는 prefix를 real token으로 사용하는 차이가 있는 것 같음 |
| P-tuning: GPT Understands, Too | Liu et al, Tsinghua | 2103 | - Discrete Prompt Engineering의 한계를 극복하기 위해 Continuous Prompt Learning의 최초 제안 <br>- we propose a novel method P-tuning  to automatically search prompts in the continuous space to bridge the gap between GPTs and NLU applications <br>- Downstream tasks : LAMA, SuperGlue Task <br>- Motivation: these models(large lm) are still too large to memorize the fine-tuning samples <br>- Prompt Encoder : bi-LSTM + 2LayerMLP(ReLU) |
| PROMPT TUNING: The Power of Scale for Parameter-Efficient Prompt Tuning | Lester et al, Google | 2104 | <br>- simplification of Prefix-tuning <br>- T5 활용한게 특징 <br>- We freeze the entire pre-trained model and only allow an additional k tunable tokens per downstream task to be prepended to the input text <br>-  we are the first to show that prompt tuning alone (with no intermediate-layer prefixes or task-specific output layers) <br>- we cast all tasks as text generation. <br>- 뭔가 방법적으로 특별한게 뭔지 모르겟음 |
| SOFT PROMPTS: Learning How to Ask: Querying LMs with Mixtures of Soft Prompts | Qin and Eisner, Johns Hopkins | 2104 | - We explore the idea of learning prompts by gradient descent—either fine-tuning prompts taken from previous work, or starting from random initialization <br>- Our prompts consist of “soft words,” i.e., continuous vectors that are not necessarily word type embeddings from the language model <br>- binary relation관계에 있는 x(입력), y(정답)을 맞추는 문제에 적용(LAMA, T-REx 등) <br>- Deep Perterbation 적용 <br>- BERT, BART 등 사용 |
| P-Tuning v2: Prompt Tuning Can Be Comparable to Fine-tuning Universally Across Scales and Tasks | Liu et al, Tsinghua | 2110 | - normal size model, hard sequence labeling task(like SQuAD)에 p-tuning 적용 <br>- prefix-tuning, soft prompt를 NLU에 적용한 버전 |
