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

- Positional Encoding에 관한 설명
  - pos= the position of the word in a sentence, i = the position of the embedding
  - ![image](https://user-images.githubusercontent.com/18374514/188919880-c7da1047-f3fa-48c6-b3e1-e98ff17624a0.png)
  - <img src="https://user-images.githubusercontent.com/18374514/188919806-0d726905-a441-489a-8c4e-867b8952b893.png" width="300">

- Whole word masking에서 주의할 점
  - whole word를 마스킹하다가 만약 masking ratio가 15%를 넘어가게 되면 다른 토큰을 원복시킬 수도 있다.
