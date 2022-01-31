---
layout: post
date: 2019-09-01 14:00
reviewed_date: 2021-09-14 14:00
title: Deep Learning for Computer Vision (EECS 498-007/598-005)
author: Justin Johnson
description: Online Lecture from University of Michigan
comments: true
math: true
category: 
- Lecture
tags:
- DeepLearning
- Computer Vision
---

Summary for Online Lecture from University of Michigan

<!--more-->

## Lecture 4. Optimization

- Optimization은 결국, Input이 Weight Matrix인 Loss 함수를 최소화 하는 W를 찾는것
  - $$w^*=argmin_wL(w)$$
- **Nemeric Gradient** : 아주 작은 h 값을 이용해서 직접 Gradient를 이용하는 방법
  - <span class='centered_small'>![2](/assets/img/michigan/2.png)</span>
  - 0.00001 이라는 아주 작은 h값에 대해서 f(x+h)-f(x) / h 를 구해서 gradient를 구하는 방법
  - W의 갯수만큼 구해야 하기 때문에 **O(Dimension)이 걸리고(느림)**, 어디까지나 **Approximate**에 불과하다.(h값의 모호함)
- **Analytic Gradient** : 실제로 미분해서 구하는 방법. Fast, exact, error-prone(오류 발생 쉬움)
  - In practice: Always use analytic gradient, but check implementation with numerical gradient. This is called a **gradient check.**
    - Torch 라이브러리 사용 가능
    - <span class='centered_small'>![3](/assets/img/michigan/3.png)</span>
    - gradgradcheck 함수는 2차 미분 확인용임
- **Gradient Decent** 
  - Iteratively step in the direction of the negative gradient(direction of local steepest descent)
  - <span class='centered_small'>![4](/assets/img/michigan/4.png)</span>
- **Stochastic Gradient Decent(SGD)**
  - 샘플링을 통해서 전체 Loss를 Expectatiion함.
  - batch 사이즈는 그렇게 critical 하게 영향을 주지는 않음.
  - 문제점 : 
    - <span class='centered_small'>![5](/assets/img/michigan/5.png)</span>
    - > Condition Number ? x의 작은 변화율에 대한 y의 변화율. 함수의 민감도 의미
  - Local minimal 혹은 Saddle point(gradient=0) 문제 발생 가능
  - 그래서 Momentum 사용
    - SGD + Momentum:
      - <span class='centered_small'>![6](/assets/img/michigan/6.png)</span>
      - v는 0으로 초기화 되었다가 이전 step의 방향으로 셋팅됨.
      - <span class='centered_small'>![7](/assets/img/michigan/7.png)</span>
      - poor conditioning 같은 경우, 지그재그로 움직이는 움직임을 좀더 smooth 하게 바꿀 수 있음(하단 그림 보면 오른쪽으로의 V덕분에 목적지에 더 빨리 도착)
        - <span class='centered_small'>![8](/assets/img/michigan/8.png)</span>
    - Nesterov Momentum: 현재 속도인 v로 한 번더 이동한 지점의 gradient를 미리 계산해서 현재 속도와 더해서 update함. 
    - > 뭔가 강화학습의 TD 개념 같기도?
    - <span class='centered_small'>![9](/assets/img/michigan/9.png)</span>
- **Adaptive Learning Rate**
  - AdaGrad(Adaptive Gradient): 변수 별 맞춤 LR. element wise 계산이기 때문에 변수별로 LR을 줄 수 있음.
    - <span class='centered_small'>![10](/assets/img/michigan/10.png)</span>
    - <span class='centered_small'>![11](/assets/img/michigan/11.png)</span>
      - 위 그림에서처럼 Elment wise로 h를 구하기 때문에 파라미터 별로 LR이 다르게 적용되고, h가 커지면 나중에는 거의 변화가 없을 수 있다.
  - RMSProp: AdaGrad에서 후반가면 너무 학습이 느려지는거 방지
    - <span class='centered_small'>![12](/assets/img/michigan/12.png)</span>
    - 최신의 Gradient와 과거의 Gradient 합 사이의 비유을 조절. 힌튼이 제안.
  - Adam : 위 두개의 장점 합침.
    - <span class='centered_small'>![13](/assets/img/michigan/13.png)</span>
    - t가 0이고 beta2=0.999이면 moment2 값이 매우 커져서 안좋은 결과를 초래 할 수 있음. 따라서 아래 그림처럼 Bias correction을 해줌.  
    - <span class='centered_small'>![14](/assets/img/michigan/14.png)</span>
- Summary
  - <span class='centered_small'>![15](/assets/img/michigan/15.png)</span>
  - **Adam** is good default choice in many cased. **SGD+Momentum** can outperform Adam but may require more tunning.
  - If you can afford to do full batch updates then try out **L-BFGS**(and don't forget to disable all sources of noise)

## Lecture 5. Neural Networks

- 선형 함수로 분류 할 수 없는 데이터의 경우, feature transform으로 해결 할 수 있으나, 이러한 함수를 찾는 것이 쉽지 않음
  - <span class='centered_small'>![16](/assets/img/michigan/16.png)</span>
- 컴퓨터 비전에서 많이 쓰이는 Feature Transform - Color histogram(단순히 어떤 색깔이 많이 등장하는가?를 표현하는듯, 위치 정보 무시)
  - <span class='centered_small'>![17](/assets/img/michigan/17.png)</span>
- 색깔은 무시하고 Edge 정보만 사용하는 Feature Transform(Representation) 과거에 object detection에 많이 사용됨.
  - <span class='centered_small'>![18](/assets/img/michigan/18.png)</span>
- 내가 가진 Data를 기반으로 Random Patch를 만들어서 Bag of image words를 만드는 방식(비교적 Flexible 하고 powerful)
  - <span class='centered_small'>![19](/assets/img/michigan/19.png)</span>
- 이러한 방식들을 사용할 때 또하나의 문제점은 이러한 다양한 Type의 Feature들을 함께 사용하는 것이 어렵다는 것.
  - <span class='centered_small'>![20](/assets/img/michigan/20.png)</span>
- 고정된 Feature Representation 위에 Score를 뽑아내는 Classifier만 학습시키는 방법은 전체 시스템의 극히 일부만 학습하는 것이므로 성능상 한계가 존재
  > 이것이 DNN의 하나의 Motivation. Feature Extraction부터 Classifier 까지 End-to-End jointly로 학습시켜보자
  - <span class='centered_small'>![21](/assets/img/michigan/21.png)</span>
- <span class='centered_small'>![22](/assets/img/michigan/22.png)</span>
- 결국 W는 이전 레이어의 입력의 영향을 다음 레이어로 **얼마나 전달 할지**를 결정하는 변수
  - <span class='centered_small'>![23](/assets/img/michigan/23.png)</span>
- Linear Classifier는 Class당 하나의 Template을 학습하고, 입력된 데이터가 이 Template들 중 어느 것에 가장 가까운지(match)를 구별하는 방식으로 Task를 수행했음
  - <span class='centered_small'>![24](/assets/img/michigan/24.png)</span>
- Neural Net의 경우도 비슷함. 첫번째 레이어는 Template을 학습하고, 두번째 레이어는 이를 recombine 함.
  - <span class='centered_small'>![25](/assets/img/michigan/25.png)</span>
  - 보통 Template을 직관적으로 해석하기는 힘듬.
  - 따라서 'Distributed Representation'으로 표현하기도 함
  - 많은 경우에, oriented edge나 opposing color가 template에 많이 포함됨.
  - 겹치는 정보도 존재 가능(Prunning 필요)
- Activation Function이 없는 경우, Layer를 쌓아도 그냥 Linear Classifier가 됨.
  - <span class='centered_small'>![26](/assets/img/michigan/26.png)</span>