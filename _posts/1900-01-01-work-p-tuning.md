---
layout: post
date: 2022-04-22 00:00
created_date: 2022-04-22 00:00
title: "[Work] KorSTS에 P-tuning 적용"
author: oglee
description: 업무 관련 내용 정리
comments: true
math: true
category:
- NLP
tags:
- DCPCSE
- SimCSE
- KorSTS
- P-tuning
---

DCPCSE 논문을 KoELECTRA에 적용해서 KorSTS 점수 확인하기
<!--more-->

### 2022-04-22

- DCPCSE 논문을 KoELECTRA에 적용해서 KorSTS 점수 확인중.
- HuggingFace 기존 코드에는 Electra에 past_key_values를 넘기는 코드가 없어서 해당 코드를 넣음. dcpcse에서는 prompt가 past_key_values로 각 레이어로 넘어가도록 되어 있기 때문
- 또, Avg pooling을 위해, 마지막 레이어에서 나온 실제 token들에 대한 last hidden들에 대해서만 avg pooling 하도록 attention_mask를 해당 길이만큼 만든 다음에 avg pooling 한다. (prompt는 avg pool에서 제외)
```python
elif self.pooler_type == "avg":
    return ((last_hidden * attention_mask.unsqueeze(-1)).sum(1) / attention_mask.sum(-1).unsqueeze(-1))
```
- 근데 점수가 너무 안좋다. 아무리 unsupervised 이지만 30점대가 나온다. 
- SimCSE의 경우, avg pooling으로 바꾸면 40점대는 나온다. 

- <u>Supervised Test</u>
  - Supervised로 테스트 하기 위해, KorNLI에서 Hard negative sample을 뽑은 3 컬럼짜리 데이터를 만들어서 학습 시킨 뒤 korSTS로 Test
  - DCPCSE는 69점까지, SimCSE는 77점까지 점수가 오름.

- <u> Finetuning 결과</u>
  - KorSTS를 KoELECTRA로 그냥 Fine-tuning 해봄
  - 80점대까지 나옴.
  - run_glue.py 코드에는 regression이면 mse score만 있어서 scikitlearn의 spearmanr 계산하는 코드를 넣음
