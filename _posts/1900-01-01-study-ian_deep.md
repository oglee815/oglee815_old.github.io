---
layout: post
date: 2021-07-03 14:00
created_date: 2021-07-03 14:00
title: "[Book] DeepLearning"
author: Ian Goodfellow, Yoshua Bengio, Aaron Courvile
description: DeepLearningBook
comments: true
math: true
category: 
- Study
tags:
- Book
- DeepLearning
---

Summary for DeepLearning by Ian GoodFellow-Chapter1
 <!--more-->

## CHAPTER 1 Introduction

오래전부터 발명가들은 생각하는 기계를 꿈꾸었고 그 역사는 고대 그리스까지 올라간다.(신화속 인물, 갈라테아나 탈로스, 판도라는 인공생명으로 간주)
이 책에서는 단어를 인식하거나 이미지에서 얼굴을 알아보는 것처럼 직관적으로 해결 할 수 있는 문제에 대한 해결책에 대해 다룬다.
그 해결책이란 기계가 개념 계통구조(hierarchy of concepts)를 이용해서 경험으로부터 배우고 세상을 이해하는 것이다. 
개념 계통구조 덕분에 컴퓨터는 간단한 개념들으 조합해서 좀 더 복잡한 개념을 배우게되고 그러한 개념들의 연결 관계를 그래프 구조로 표현한다면, 여러 층(layer)으로 이루어진 '깊은' 그래프가 나올 것이다. 이 때문에 이러한 인공지능의 접근 방식을 **심층 학습(Deep Learning)**이라 부른다.

인공지능의 초기 성공 사례 중 다수는 체스처럼 비교적 단조롭고 형식적인 환경(relatively sterile and formal environments)에서 일어난 것으로, 컴퓨터가 실세계에 관한 지식을 많이 갖출 필요가 없었다(did not require computers to have much knowledge aboutthe world).

하지만 인공지능이 진정으로 지능적으로 행동하려면 비형식적 지식(informal knowledge)을 컴퓨터에 제공해야 한다.

세계에 관한 지식을 형식 언어(formal language)를 이용해서 하드 코딩하는 식의 지식 베이스(knowledge base) 접근 방식은 성공적이지 못했다.
그 과정에서 원본 자료에서 패턴을 추출해서 스스로 지식을 획득하는 능력이 필요함을 알게 되었다. 그러한 능력을 **기계 학습(Deep Learning)**이라고 부른다. 

이러한 알고리즘의 성과는 주어진 자료의 표현(representation)에 크게 의존한다. 사람들은 아라비아 숫자로 된 수들에 관한 계산은 쉽사리 해내지만 로마 숫자로 된 수들에 관한 계산에는 시간을 훨씬 많이 소비한다. 자료를 어떻게 표현하느냐에 따라 기계 학습 알고리즘의 성능이 크게 달라진다는 것은 놀랄 일이 아니다. 

인공지능 과제 중에는 주어진 과제에 맞는 적절한 특징 집합을 추출하고 그 특징을 알고리즘에 적용해서 풀 수 있는 경우가 많다. 하지만 그러한 특징을 추출해 내는 것은 어려운 일이다. 이 문제에 대한 해법은 표현을 출력으로 사상하는 방법뿐만 아니라 표현 자체까지도 인공지능 시스템이 기계 학습 알고리즘으로 배우게 하는 것이다(not only the mapping from representation to output but also the representation itself).

표현 학습 알고리즘의 대표적인(quintessential) 예는 **AutoEncoder**이다. 오토인코더는 입력 자료를 어떤 다른 표현으로 변환하는 encoder 함수와 그 표현을 원래의 자료 형식으로 되돌리는 decoder 함수의 조합이다.
오토인코더는 입력이 인코더를 거쳐 디코더를 통과하는 과정에서 정보를 최대한 유지하도록 훈련되며, 그와 함께 새로운 표현이 여러 가지 좋은 속성을 가지도록 훈련되기도 한다.

When designing features or algorithms for learning features, our goal is usually to separate **the factors of variation** that explain the observed data. In this context, we use the word "factors" simply to refer to separate sources of inﬂuence;the factors are usually **not combined by multiplication**. Such factors are often not quantities that are directly observed. Instead, they may exist as either unobserved objects or **unobserved forces** in the physical world that affect observable quantities. They may also exist as constructs(구인構因) in the human mind that provide useful simplifying explanations or inferred causes of the observed data. 녹음된 음성을 분석할 때는, 이를 테면 화자의 나이, 성별, 억양, 발음된 단어 등이 변동 인자일 수 있다.

여러 실세게 인공지능 응용의 주된 난제는 관측 가능한 자료에 영향을 주는 변동 인자(factors of variation)가 너무 많다는 것이다. 대부분의 인공지능 응용에는 변동 인자들을 **풀어헤쳐서(disentangle)**, 주어진 과제와 무관한 변동 인자들을 골라내는 능력이 필요하다.

물론 그런 고수준의 추상적인 특징들을 원본 자료로부터 추출하는 것이 아주 어려울 수 있다. **문제를 풀기 위해 표현을 얻는 것이 애초의 문제를 푸는 것만큼이나 어려울 때는 이러한 표현 학습이 별 도움이 되지 않는 것으로 보인다.**

심층 학습은 자료를 다른 좀 더 간단한 표현들을 이용해서 표현함으로써 표현 학습의 이러한 문제를 해결한다.

## CHAPTER 2 선형대수

- broadcasting : 스칼라 값을 행렬이나 벡터에 곱하거나 더할 때 모든 value에 더해지도록 암묵적 복사가 되는거
- 그냥 성분끼리 곱하는 연산 = 성분별 곱(element-wise product) 또는 아다마르 곱(Hadamard product)  A⊙B 으로 표시
- 내적은 dot product
- 두 벡터의 내적은 교환 법칙을 만족. $$x^Ty=y^Tx$$
- 일차연립 방정식(선형 방정식)을 다음과 같이 행결 벡터로 표현 가능
- <strong>$$Ax=b$$</strong>
- 방정식의 해가 몇 개인지 분석할 때는, A의 열들을 원점(origin: 모든 성분이 0인 벡터에 해당하는 점)에서부터 나아 갈 수 있는 서로 다른 방향들을 명시한다고 간주하고, 그 방향들로 나아가서 **실제로 b에 도달하는 방법**이 몇 개인지 센다. 이러한 방식에서 x의 각 성분은 해당 방향으로 얼마나 멀리 나아가야 하는지를 나타낸다. **즉, 각 $$x_i$$는 열 i의 방향으로의 진행 거리를 나타낸다.**
- <span>$$Ax=\sum_{i}x_{i}A_{:,i}$$</span>
  - 일반화하자면, 이런 종류의 연산을 **일차결합**(linear combinations, 또는 선형 결합)이라고 부른다.
  - 주어진 벡터 집합의 **일차결합으로 얻을 수 있는 모든 점의 집합을 그 벡터 집합의 생성공간(span; 또는 펼침)이라고 부른다.**
- 일차 연립방정식 Ax=b 에 해가 있는지는 b가 A의 열들의 **생성공간**에 속하는지로 판정할 수 있다. 그러한 특정한 생성공간을 **A의 열공간(column space)** 또는 치역(range)이라고 부른다.
- 따라서 모든 b에 대해 일차연립방정식 Ax=b에 하나의 해가 존재하려면, A의 열공간이 $$\mathbb{R}^m$$ (m은 A의 행)전체와 같아야 한다. 만일 $$\mathbb{R}^m$$의 어떤 점이 그 열공간에 존재하지 않는다면, 그 점에 해당하는 b에 대해서는 방정식에 해가 존재하지 않을 가능성이 있다.
- A의 열공간이 $$\mathbb{R}^m$$ 전체와 같아야 한다는 요구조건은 A의 열이 적어도 m개여야 한다는, **다시 말해 n>=m 이어야 한다는 요구조건으로 이어진다.**
- 
