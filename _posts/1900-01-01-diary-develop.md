---
layout: post
date: 2022-04-13 00:00
reviewed_date: 2022-04-13 00:00
title: "[Diary] Daily log"
author: oglee
description: 업무 관련 내용 정리
comments: true
math: true
category:
- Diary
tags:
- 개발자 일기
- 연구원 일기
---

일하면서 걍 매일매일 주절주절
 <!--more-->

### 2022-04-13 수

DCPCSE 논문을 KoELECTRA에 적용해서 KorSTS 점수 확인중.. 근데 점수가 너무 안좋다. 
아무리 unsupervised 이지만 30점대가 나온다.
SimCSE의 경우, avg pooling으로 바꾸면 40점대는 나온다.
근데 DCPCSE는 avg pooling 할 경우 문제가 생긴다.
<u>왜냐하면, 기존 코드는 attention_mask 길이가 prompt를 제외한 실제 입력 토큰의 길이만큼 avg를 때리는데, 여기에 prompt까지 넣어서 함께 avg pooling을 해야한다.</u>
근데 bert의 최종 output은 prompt가 없는 실제 입력 토큰의 last_hidden만 나오도록 구현이 되어있다.
이걸 어떻게 해결해야 하나?
