---
layout: post
date: 2022-08-22 00:00
created_date: 2022-08-22 00:00
title: "[work] huggingface, EarlyStoppingCallback 사용시 주의사항"
author: oglee
description: "EarlyStoppingCallback 사용시 주의사항"
comments: true
math: true
category:
- Work
tags:
- Huggingface
- EalryStoppingCallback
---

huggingface, EarlyStoppingCallback 사용시 주의사항

<!--more-->

- 기본 인자 셋팅을 잊지 말 것
  -  early_stopping_patience: int = 1, early_stopping_threshold: Optional[float] = 0.0)
- 본인이 Trainer를 직접 만들었다면, 그리고 evaluate 함수까지 직접 구현했다면 반드시 <strong>self.callback_handler.on_evaluate</strong> 을 호출해줄것
