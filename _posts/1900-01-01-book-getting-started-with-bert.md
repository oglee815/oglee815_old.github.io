---
layout: post
date: 2022-09-08 00:00
created_date: 2022-09-08 00:00
title: "[Book] Getting Started with Google BERT"
author: oglee
description: "Getting Started with Google BERT"
comments: true
math: true
category:
- book
tags:
- NLP
- BERT
---

Getting Started with Google BERT by Sudharsan Ravichandiran
<!--more-->

#### Positional Encoding에 관한 설명
- pos = the position of the word in a sentence, i = the position of the embedding
- ![image](https://user-images.githubusercontent.com/18374514/188919880-c7da1047-f3fa-48c6-b3e1-e98ff17624a0.png)
- <img src="https://user-images.githubusercontent.com/18374514/188919806-0d726905-a441-489a-8c4e-867b8952b893.png" width="300">

#### Whole word masking에서 주의할 점
- whole word를 마스킹하다가 만약 masking ratio가 15%를 넘어가게 되면 다른 토큰을 원복시킬 수도 있다.

#### BERT의 pretraining 시 하이퍼파람
- 배치 256, 100M steps, lr=1e-4, B1=0.9, B2=0.999, warmup=10K
- 0.1 dropout, GELU Activation.
- <img src="https://user-images.githubusercontent.com/18374514/188924033-29df04ec-918a-4808-a7ec-cc243bfc6956.png" width="300">

#### Byte Pair Encoding vs Byte-level byte pair encoding vs WordPiece
- Byte-pair Encoding 
  - 만약 우리 corpus에 다음과 같은 단어들이 있다고 가정하자, cost는 등장 횟수
  - <img src='https://user-images.githubusercontent.com/18374514/189703095-f8f9d1de-4ebc-450c-a892-43e4919ebfd8.png' width='300'>
  - 그리고 우리 vocab size를 14개로 정했다고 가정.
  - 1) 모든 unique character를 vocab에 추가
    - <img src='https://user-images.githubusercontent.com/18374514/189703595-eec2197d-0294-4d22-b2dc-3f49c675bb9f.png' width='300'>
  - 2) 그럼, vocab 이 11개가 되므로, 3개가 부족함. 3개를 더 추가해야함.
  - 3) 등장 횟수 순서로 보면, st와 me가 각각 4번씩 등장하므로, vocab에 추가함    
  - 4) 그 다음에, me+n이 2번 등장하므로 men 도 추가
    - <img src='https://user-images.githubusercontent.com/18374514/189704532-9be2b8c5-0474-43c9-b320-9d0787c115ea.png' width='300'>
  - 5) 그럼 최종적으로 아래처럼 됨.
    - <img src='https://user-images.githubusercontent.com/18374514/189704758-d891f635-5be4-43bf-99bc-67e65a18fbb6.png' width='300'>

- Byte-level byte pair Encoding(BBPE)
  - Character를 Byte로 변경함, 그 후 BPE 실행
  - ex) best -> 62 65 73 74
  - Multilingual settting에서 장점이 크다.

- WordPiece
  - BPE와 기본적으로 비슷하나, frequency 기반으로 merge하지 않고 likelihood에 기반함
  - train set을 대상으로 language model를 우선 학습해서(통계적), maximum likelihood를 갖는 char 조합을 merge함.
- 
