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
- For harmlessness, we invite crowdworkers to adversarially probe or <r>‘red-team’ our language models in order to provoke harmful responses</r>
- RLHF를 하면 대부분의 NLP Tasks에서도 괜찮은 성능을 보인다(그러나 모델 사이즈가 작으면 성능 감소 있음)
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/01e70b3a-79b4-4646-9495-f55a8f4ef1ec)

## Contributions
- Dialogue Preference Datasets : helpfulness and harmlessness data, three tranches of data(one from initial models, one with rejection sampling against early preference models, one with models trained with online RLHF
- Alignment with Human values has many benefits and essentially no cost to performance
  - Smaller models experience severe 'alignment taxes'(13B, 52B에서는 오히려 NLP task도 잘함)
  - 요약 테스크와 관련한 specialized skill을 HH 데이터와 함께 학습하니, 둘다 잘하더라
  - There is a tension between helpfulness and harmlessness
  - one can use OOD(out-of-distribution) Detection techniques to reject most strange and harmful request
- Scaling, RLHF Robustness, and Iterated 'Online' Training
  - We study scaling relations for PM accuracy as a funtion of model and dataset size, and find rougle log-linear trends
  - robustness RLHF, <r>larger PMs are more robust than smaller PMs</r> 
