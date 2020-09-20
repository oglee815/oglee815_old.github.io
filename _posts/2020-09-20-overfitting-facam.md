---
layout: post
date: 2020-09-20 23:44
title: Overfitting?
author: 이경택 강사
description: Overfitting 에 대한 가벼운 설명
comments: true
math: true
category: 
- Fastcampus
tags:
- Machine Learning
- 머신러닝과 데이터분석 A-Z
---

 <!--more-->
![overfitting1](/assets/img/overfitting1.png)

#### 과적합이란?

- Train에만 Fit 되서 Test에 성능이 안좋은 경우
- 모형이 복잡할 수록, 데이터가 적을 수록 잘 발생
- 데이터가 많으면 과적함은 잘 일어나지 않음
- Overfitting 을 완전히 없애는 방법은 존재하지 않는다.
  
![overfitting2](/assets/img/overfitting2.png)

- Y를 정답, Y^를 예측치, 입실론을 맞출 수 없는 에러로 본다면, 
- Y = f(X) + e, Y^ = f^(X)
- Var(f^(X)) -> 모델의 예측치가 얼마나 퍼져있나
- Bias(f^(X)) -> 모델의 예측치가 정답과 얼마나 떨어져 있나
- Var(e) -> 맞출 수 없는 에러


![overfitting3](/assets/img/overfitting3.png)

- Bias와 Variance 는 Trade Off 관계가 있기 때문에 선택을 해야 하는데
- **앙상블 러닝이 사실은 Low Bias, High Variance**를 해결 할 수 있다. 왜냐면 모델들이 Varicance 가 크다고 해도 앙상블을 통해 평균을 내는 등의 방식으로 학습하면 가운데 과녁을 맞출 수 있다.