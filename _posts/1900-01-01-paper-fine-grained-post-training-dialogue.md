---
layout: post
date: 2022-12-04 14:00
created_date: 2022-12-04 14:00
title: "[Paper] 20?21? Fine-grained Post-training for Improving Retrieval-based Dialogue
"
author: Janghoon Han, Taesuk Hong, Byoungjae Kim, Youngjoong Ko, Jungyun Seo @ 서강대, 성균관대, LG AI research
description: Fine-grained Post-training for Improving Retrieval-based Dialogue
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Diaogue
- Response Selection
- Response Retrieval
---

Random, same context, right answer 세개의 Label을 맞추는 post-training 방식으로 Retrieval-based dialogue sytem 성능 개선
<!--more-->

# Key idea
- ![image](https://user-images.githubusercontent.com/18374514/205480399-024b80b8-4d3b-4c0d-9f1f-07e7ef1c2f4e.png)
- Short Context-Response: Short Conext(2-4)길이의 Utterance 이후에 Resoponse가 correct인지 아닌지(0, 1)
- Utterance Relevance Classification: Short Context 이후에 Reponse가 Random, same context, correct 중 하나를 맞추는 문제(정답 3개)
- Loss = Loss_MLM + Loss_URC
- **버트의 NSP와 알버트의 SOP를 함께 하면 어떨까 하는 것에 모티베이션**이 큰 듯함.

# 궁금증/아쉬운 점
- SCR 과 URC를 함께 했다는건지, SCR를 하고 URC를 했다는건지?
- SCR과 URC를 진행하는데 있어서 많은 디테일이 없는 것 같음. epoch이나 lr이나.. 
- post-training 이니까 SCR이나 URC를 하고 Fine-tuning을 했다는 말인가? 왜 이런 것에 대해 설명이 없는지? 내가 못찾은건가
- NSP_SCR이랑 URC_SCR이랑 의외로 큰 점수 차이가 안남
