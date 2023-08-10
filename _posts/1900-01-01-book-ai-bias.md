---
layout: post
date: 2023-06-29 14:08
created_date: 2023-06-29 14:08
title: "[book]인공지능의 편향과 챗봇의 일탈"
author: 정원섭
description:
comments: true
mathjax: true
category:
- book
tags:
- 챗봇
- 편향성
---

인공지능의 편향성에 관한 책
<!--more-->
<mark style='background-color:pink'>
<style>
r { color: Red }
o { color: Orange }
g { color: Green }
</style>


# 인공지능의 편향과 챗봇의 일탈
- 2016년 3월에 발표된 MS의 Tay 사태 -> 알파고 이후에 일어난 일이므로 사람들에게 큰 영향을 끼침.
- 2021년 이루다 사태, 성노리개나 성 소수자 발언 등 -> 심각한 물의를 일으킴

## 데이터에서 나타날 수 있는 편향
### 데이터가 만드는 인공지능
- 인공지능을 똑똑한 인공지능으로 학습시키기 위한 최초의 과정이자 가장 노력이 많이 소요되는 작업은 바로 양질의 훈련 데이터를 확보하는 것
- 데이터의 객관성은 데이터가 지향하는 목표를 달성하기 위해 필요한 조건을 만족하는 객관적인 데이터 습득을 전제해야 하며, 이 과정에서 당연히 <r>주관적인 개입은 배제</r>
- 주관적 개입이라 함은, 연구자의 임의적인 의사결정 혹은 태업에 의한 부도덕한 일탈 행위라기 보다, 복잡한 <r>실험과정과 데이터를 처리하는 과정</r>이 야기하는 데이터 연구자의 한계 정도로 이해
- 이는 인간공학적 관점엥서는 '인간 정보처리시스템의 정보처리용량의 한계'로 표현되기도 함.
`편향성이 버그에 가깝다는 건가?`

### 학습 데이터의 양적 성장
- 인공지능이 의미 있는 성능을 갖추기 위해 다량의 학습 데이터가 필요하다는 점은 이제 상시거럼 되었음.
- 1998년 얀 르쿤 교수가 공개한 엠니스트의 데이터 세트를 들 수 있음.
- 그리하여 미국에서는 오바마 정부가 공공데이터를 확보하기 위하여 공유 플랫폼(data.gov)를 시작하였고, 우리 나라도 공공데이터 포탈(data.go.kr), 한국정보화진흥원(NIA) 등이 있음.

### 학습 데이터의 품질
- 클라우드 서비스 품질 모델 표준안인 ISO/IEC 25012에서는 소프트웨어 품질에 영향을 미치는 데이터 품질 모델을 '데이터의 고유 성질'과 '소프트웨어 종속적 성질'로 나누고 있다. 여기서 데이터의 고유 성질에는 **정확성, 완전성, 일관성, 신뢰성, 현재성'**이 있다고 함.
  - `Instruction tuning 시에도 데이터의 품질이 중요함을 뼈저리게 느끼고 있음. 특히 정확성, 일관성, 신뢰성 등은 할루시네이션과 매우 밀접한 영향을 끼치는 것으로 느껴짐`
- 소프트웨어 종속적 성질에는 '가용성, 이식성, 복구성, 유용성, 적합성, 기밀성, 효유성, 정밀성, 추적성, 이해성'이 있다고 말함. 
- 그러나 대부분 정량적 품질을 특정하기 어려움.