---
layout: post
date: 2022-04-22 00:00
created_date: 2022-04-22 00:00
title: "[Work] P-tuning Dialogue"
author: oglee
description: P-tuning Dialogue
comments: true
math: true
category:
- NLP
tags:
- DCPCSE
- PSP
- Dialogue Summarization
- P-tuning
---

Prompt 를 활용한 Dialogue Summarization
<!--more-->

# Dataset
- Samsum
  - 전처리
    - load_dataset('json', data_files='file') 형태로 로드하면 에러나 발생해서 아래처럼 바꿈(그리고 load_datset에 filed='data' 추가)
      - 기존 형태 : ![image](https://user-images.githubusercontent.com/18374514/169343056-4ffb8711-8cec-4bfe-9801-57a78c19f328.png)
      - 바뀐 형태 : ![image](https://user-images.githubusercontent.com/18374514/169343508-0461d52a-6253-42d2-a85a-041605ae82ef.png)

# 코드
- transformer 새로운 코드로 시작
