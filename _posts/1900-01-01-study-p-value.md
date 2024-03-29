---
layout: post
date: 2022-06-14 00:00
created_date: 2022-06-14 00:00
title: "[통계] 모평균 추정시 P(X>t) or P(X<t)?"
author: oglee
description: 통계에서 헷갈리는 개념
comments: true
math: true
category:
- Study
tags:
- 통계
- t-test
- 모평균추정
---

Null 가정에 따른 부등호의 방향! P(X>t) or P(X<t)
<!--more-->

- 샘플로부터 모평균의 추정을 할 경우, 일반적으로 다음과 같은 영가설을 세우게 된다.
  1. μ = μ0 (μ0는 어떤 특정한 값)
  2. μ < μ0
  3. μ > μ0
 
- 그럼 위 가설들을 검증하기 위해 p-value를 구하게 된다. 구한 p-value가 높으면 영가설을 채택한다.
- 이 때, P-value는 다음 중 하나이다.
  1. P(X>t)
  2. P(X<t)
  3. 2 * P(X > \|t\|)
- t-statistics는 (샘플 평균 - μ0)로 이루어져 있기 때문에 \|t\|이 크면 μ0와 어쨌든 차이가 크다고 볼 수 있다.
- 이 때, 어떤걸 선택해야 하는지 헷갈린다. μ = μ0는 μ가 μ0보다 너무 커도 안되고 너무 작아도 안되기 때문에 3번인 2 * P(X > \|t\|) 이 p-value인 것을 쉽게 알 수 있다.
- μ < μ0 를 검증하고 싶으면 P(X<t)가 p-value일 것 같은데 그렇지 않다.
  - 곰곰히 생각해보면, μ0가 어떤 값이든, μ가 엄청 커서 9억 9천 9백만 쯤 되버린다고 상상해보면(그니까 worse data) μ < μ0 가 일어날 확률은 갈 수록 작아진다. 즉 p값은 매우 작아질 수 밖에 없다.
  - ![image](https://user-images.githubusercontent.com/18374514/173586822-b83df71a-8a7c-4d2e-a8ec-6300c55e571b.png)
  - 위 그림에서처럼 P(X<t)는 너무나 큰 값 0.9999 이 되기 때문에 검증할 가치가 없어진다. 
  - 따라서 P(X>t) 를 p-value로 생각해서 구할 수 있고, 이 값은 매우 작을 것이기 때문에 영가설은 기각이되고 9척 9천 9백만 이라는 큰 값은 가진 μ는 μ0보다 크다고 볼 수 없다고 판명한다.
- 위와 반대로 μ > μ0를 검증하고 싶으면 아주 작은 값(그니까 worse data)를 상상해보면,
  - ![image](https://user-images.githubusercontent.com/18374514/173588148-c6d2554b-d277-4f0a-ba61-6f5468b247fe.png)
  - 위 그림처럼 되는데, 그럼 우측의 P(x>t) 는 매우 큰 값이 나온다. 그럼 검증에 사용될 가치가 없다.
  - 따라서 좌측의 P(X<t)를 p-value로 사용하게 된다.
- **결론적으로, worse data(아주 크거나 아주 작은 값)를 기준으로 했을때 P(X>t)와 P(X<t) 중 어떤 것을 사용하는게 적절한지 조금만 생각해보면 된다.**

