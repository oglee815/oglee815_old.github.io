---
layout: post
date: 2021-11-09 14:00
reviewed_date: 2022-02-23
title: Learning to Draw, Emergent Communication through Sketching(NeurIPS 2021)
author: Daniela Mihai, Jonathon Hare @The University of Southampton
description: Learning to Draw
comments: true
math: true
category: 
- Paper
tags:
- Computer Vision
- Hinge Loss
---

In this work, we explore a visual communication channel between agents that are allowed to draw with simple strokes.
<!--more-->

## Abstract
> Evidence that visual communication preceded written language and provided a basis for it goes back to prehistory, in forms such as cave and rock paintings depicting traces of our distant ancestors. Emergent communication research has sought to explore how agents can learn to communicate in order to collaboratively solve tasks. Existing research has focused on language, with a learned communication channel transmitting sequences of discrete tokens between the agents. In this work, we explore a visual communication channel between agents that are allowed to draw with simple strokes. Our agents are parameterised by deep neural networks, and the drawing procedure is differentiable, allowing for end-to-end training. In the framework of a referential communication game, we demonstrate that agents can not only successfully learn to communicate by drawing, but with appropriate inductive biases, can do so in a fashion that humans can interpret. We hope to encourage future research to consider visual communication as a more flexible and directly interpretable alternative of training collaborative agents.

## Summary
- 다루는 문제
  - referential communication game
    - Imagine you and a friend are playing a game where you have to get your friend to guess an object in the room by you sketching the object. No other communication is allowed beyond the sketched image.(캐치마인드 같은 게임?)
  - This paper explores **how artificial agents parameterised by neural networks can learn to play similar drawing games.**
  - More specifically, we reformulate the traditional referential game such that <u>one agent draws a sketch of a given photo and the second agent guesses</u>, based on the drawing, the corresponding photo from a set of images.
- Contributions
  - We leverage recent advances in differentiable sketching that enables us to construct an agent that **can learn to communicate intent through drawing**
  - Introducing **a perceptual loss** improves human interpretability of the communication protocol, at little to no cost in the gameplay success;
  - Changes to the game objective, such as playing an object-oriented game, can steer the emergent communication protocol towards a more **pictographic(그림 문자) or symbolic form of expression**;
  - Inducing a shape-bias into the agents’ visual system leads to more explainable drawings;
  - A drawing agent trained with a perceptual loss can successfully communicate and play the game with a human.

# A model for learning to communicate by drawing
- Two agents : sender and receiver
  - the sender learns to draw by playing a game with the receiver
- Overview
- <span class='centered'>![overview2](/assets/img/learning_to_draw/overview2.jpg)</span>
  - Sender가 그림을 입력으로 받고 스케치를 해서 Receiver에게 주면 Receiver는 전달 받은 스케치와 여러 그림 중 정답 그림을 맵핑하는 작업을 통해 학습
- Loss Function
  - <span>![loss](/assets/img/learning_to_draw/loss.JPG)</span>
  - where <u>x is the score vector</u> produced by the receiver, and <u>y is the true index of the target</u>
  - 정답이 아닌 j에 대한 score xj는 낮을수록 좋고, 정답 y에 대한 score인 xy는 높이되 값이 커져서 우측 값이 음수가 되면 0을 loss로 취함<a class="color_a" href="https://www.youtube.com/watch?v=KT4iD6yiqwo&list=PL1Kb3QTCLIVtyOuMgyVgT-OeW0PYXl3j5&index=2">(설명 링크)</a>
  - hinge loss 설명
    - <span>![hinge](/assets/img/learning_to_draw/hinge.JPG)</span>
    - i == j 를 제외하지 않고 loss에 포함시키면 max( 0, 1) 이 loss에 추가되어 결과적으로 loss가 1이 증가하게 된다.
    - sum 대신 mean을 사용해도 되지만 어차피 loss minimize 관점에서는 똑같다.
  - Perceptual Loss
    - <span>![loss2](/assets/img/learning_to_draw/loss2.JPG)</span>
    - <span>![loss3](/assets/img/learning_to_draw/loss3.JPG)</span>