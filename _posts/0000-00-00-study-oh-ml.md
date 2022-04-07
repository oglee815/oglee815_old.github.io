---
layout: post
date: 2017-12-05 00:00
reviewed_date: 2022-02-06 14:00
title: Machine Learning
author: 오일석
description: Machine learning
comments: true
math: true
category:
- Study
tags:
- Book
- Machine Learning
---

Machine learning by 오일석
 <!--more-->

## 확률통계

### Conditional Independent

- <span>$$P(x,y|z)=P(x|z)P(y|z)$$</span> 이면 z가 주어졌을때 x와 y는 조건부 독립

### 베이즈 정리(Bayes' theorem)

- 일반적으로 x와 y가 같이 일어날 결합확률(동시에 일어날 확률)이나 y와 x가 같이 일어날 결합확률이 같다.
  - <span>$$P(x,y) = p(y,x)$$</span>, 그러므로 아래 식이 성립
  - <span>$$P(y,x) = P(x|y)P(y) = P(x,y) = P(y|x)P(x)$$</span>
- 위 식을 정리하면 베이즈 정리
  - $$P(y|x) = \frac{P(x|y)P(y)}{P(x)} $$
- 이때, P(y\|x)는 x라는 사건 이후에 발생했으므로 사후확률(posterior probability),
- P(y)는 x와 상관 없이 알 수 있으므로 사전 확률(prior probability)라고 한다.
- <span class="my_highlight">P(x\|y)는 우도(likelyhood)라고 한다.</span>
- 분모의 P(x)는 기계학습에서 무시된다. 정답중에서 상대적인 확률만 알면되기 때문.

#### 베이즈 정리를 기계학습에 적용

- 결국 기계학습에서 알고자 하는건 입력 x가 들어왔을 때, 가장 정답일 확률이 높은 y를 찾는 것이다.
  - <span>$$\hat{y}=argmax_y\,P(y|x)$$</span>
- 그러나 입력 x가 만드는 공간은 무수히 많으므로 p(y\|x)를 바로 추정하는 것은 어려움.
- 따라서 베이즈 정리를 통해 우회적으로 문제를 해결.
  - <span class="my_highlight">사전확률 P(y)와 우도 P(x\|y)를 구할 수 있으면 사후확률 P(y\|x)</span>를 구할 수 있다.
- 사전확률 P(y)는 랜덤샘플링 한 결과에서 y의 분포를 보면 됨.
- 우도 P(x\|y)는 y가 고정되어 있는 상태에서 x의 분포를 추정하는데, y가 고정되어 있으므로 각 y에 대해서 독립적으로 계산이 가능.
- 우도 추정에 적용할 수 있는 여러 가지 확률밀도 추정(density estimation) 방식이 있는데, <span class="my_highlight">가우시안, 파젠 창, 가우시안 혼합</span> 등 여러 방법 존재

#### 최대 우도

- <span>$$\mathbb{X}=\{x_1, x_2, ..., x_n\}$$</span>
- From [https://angeloyeo.github.io/2020/07/17/MLE.html](https://angeloyeo.github.io/2020/07/17/MLE.html)
  - likelihood(분포Y\|데이터B) 는 얻은 데이터 B가 추정하고자 하는 분포 Y에서 나왔을 확률이다.
  - 아래와 같이 전체 표본집합의 결합확률밀도 함수를 likelihood function이라고 한다.
  - $$P(\mathbb{X}|\theta)=\prod_{k=1}^{n}P(x_{k}|\theta)$$
  - (각 샘플들이 IID 이기 때문에 곱해줌)
  - 보통 계산의 편의성 및 확률이라는 작은 값을 n번 곱하면 매우 작은 수가 되므로 log likelyhood function을 계산함
    - $$L(\theta| \mathbb{X}) = \log P(\mathbb{X}|\theta) = \sum_{i=1}^{n}\log P(x_i | \theta)$$
  - 결국 <span class="my_highlight">Maximum Likelihood Estimation은 Likelihood 함수의 최대값을 찾는 방법</span>이라 할 수 있다.
  - <span>$$\hat{\theta}=argmax_{\theta}\;log\;P(\mathbb{X}|\theta)$$</span>
  - log 함수는 단조증가 함수이기 때문에 likelihood function의 최대값을 찾으나 log-likelihood function의 최대값을 찾으나 두 경우 모두의 최대값을 갖게 해주는 정의역의 함수 입력값은 동일하다. 따라서 log를 취해도 solution은 같다.
  - 어떤 함수의 최대값을 찾는 방법 중 가장 보편적인 방법은 미분계수를 이용하는 것이다.
  - 즉, 찾고자하는 파라미터  θ 에 대하여 다음과 같이 편미분하고 그 값이 0이 되도록 하는  θ 를 찾는 과정을 통해 likelihood 함수를 최대화 시켜줄 수 있는  θ 를 찾을 수 있다.
  - $$\frac{\partial}{\partial \theta}L(\theta|x) = \frac{\partial}{\partial \theta}\log P(x|\theta) = \sum_{i=1}^{n}\frac{\partial}{\partial\theta}\log P(x_i|\theta) = 0$$


## 정보이론

- 특정 사건에 대한 메시지가 가진 정보량을 수량화 하는 방식(e.g, 고비사막에 눈이 왔다.)

### 자기 정보(Self-information)

- 확률변수를 x라 하고 x의 정의역이 $$\{e_1, e_2, ..., e_k\}$$
- 사건 $$e_i$$의 정보량은 $$h(e_i)$$ = 자기 정보(self-information)라 한다.
- $$h(e_i)=-log_{2}P(e_i)$$ or $$h(e_i)=-log_{e}P(e_i)$$
- 밑이 2인 로그함수는 자기 정보의 단위가 <span class="my_highlight">bit</span>
- 밑이 e인 로그함수는 자기 정보의 단위가 <span class="my_highlight">nat(나츠)</span>

### 엔트로피(entropy)

- 확률분포의 무질서도 또는 불확실성을 측정
- $$H(x) = -\sum_{k=1}^{K} P(x_k) log_2 P(x_k)$$
- $$H(x) = \int_{-\infty}^{\infty}P(x)log_2P(x)$$
- 윷놀이보다 주사위의 엔트로피가 더 큰데, 주사위는 모든 사건이 동일한 확률을 가지기 때문에 불확실성이 높기 때문이다.
- 정의역이 커지면 엔트로피도 커진다.

### 교차 엔트로피

- 서로 다른 두 확률분포 P와 Q사이의 교차 엔트로피를 구해야 할 때도 있다.
- $$H(P, Q) = -\sum_{x}P(x)lnQ(x)$$
- 위 식을 아래와 같이 유도 가능
- <span>$$H(P, Q) = -\sum_{x}P(x)lnQ(x)$$</span><br><br>
<span>$$= -\sum_{x}P(x)lnP(x) + \sum_{x}P(x)lnP(x)-\sum_{x}P(x)lnQ(x)$$</span><br><br>
<span>$$=H(P) + \sum_{x}P(x)ln\frac{P(x)}{Q(x)}$$</span>
- <span class="my_highlight">마지막 식의 두 번째 항을 KL Divergence라고 한다.</span>
- 즉, <span class="my_highlight">P와 Q의 교차엔트로피는 P의 엔트로피 + P와 Q간의 KL 다이버전스</span> 이다.
  - $$KL(P||Q) = \sum_{x}P(x)ln\frac{P(x)}{Q(x)}$$
  - P와 Q가 같으면 0이다. 따라서 거리 개념을 내포한다.
  - 하지만 $$KL(P\|Q) != KL(Q\|P)$$ 이므로 엄밀한 수학적 정의에 따르면 거리가 아니다.
