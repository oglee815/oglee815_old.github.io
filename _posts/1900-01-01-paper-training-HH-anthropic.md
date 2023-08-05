---
layout: post
date: 2023-08-05 14:08
created_date: 2023-08-05 14:08
title: "[paper] Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback"
description: Anthropic RLHF paper
comments: true
mathjax: true
category:
- Paper
tags:
- Anthropic
- ChatGPT
- RLHF
---

Anthropic's RLHF
<!--more-->

<!-- <mark style='background-color:pink'> -->
<style>
r{color:Red}
o{color:Orange}
g{color:Green}
</style>

# Abstract
- We apply preference modeling and <r>reinforcement learning from human feedback (RLHF)</r> to finetune language models to act as helpful and harmless assistants.
- We find this alignment training improves performance on almost <r>all NLP evaluations</r>, and is fully compatible with training for specialized skills such as <r>python coding and summarization.</r>
- We explore an iterated online mode of training, where preference models and RL policies are updated on a <r>weekly -cadence</r> with fresh human feedback data, 
- Finally, we investigate the robustness of RLHF training, and identify a roughly <r>linear relation between the RL reward and the square root of the KL divergence between the policy and its initialization</r>
> 그니까, 학습 된 모델이 초기 모델과 KL이 멀어질수록, RM의 reward가 높아짐

# Introduction
- In this paper we show that we can train a relatively helpful and harmless (HH) natural language assistant by collecting human preference data and applying the techniques of <r>preference modeling (PMing) and reinforcement learning from human feedback (RLHF)</r>
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/b22e6aca-69f4-4b18-89f8-41fb0b9574d0)
- 
