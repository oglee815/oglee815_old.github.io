---
layout: post
date: 2023-08-09 14:08
created_date: 2023-08-09 14:08
title: "[paper] LLAMA2"
description: LLAMA2
comments: true
mathjax: true
category:
- Paper
tags:
- Facebook
- ChatGPT
- LLAMA2
- RLHF
---

LLAMA2 paper

<!--more-->

<style>
r{color:Red}
o{color:Orange}
g{color:Green}
</style>

# Introdution
- 7~70B, LLAMA2, LLAMA2-Chat(dialogue use case) 공개

# Pretraining
- 2 Trillion tokens
- LLAMA1 architecture, per-normalization usning RMSNorm, SwiGLU activation, rotary positional embeddings
- The primary architectural differences from Llama 1 include <r>increased context length and grouped-query attention</r>
- AdamW, cosine learning rate warmup 2000.

# Fine-tuning
- 아 신기하게 FLAN으로 SFT를 만듦. 하지만 이경우 diversity and quality가 불충, 사람이 직접 수천개의 레이블링 작업
- 사실, 훨씬 많은 데이터를 만들었으나, 양보다 퀄러티가 좋다는 것을 알게 되어 수천개만 사용
- b64, 4096 msl, 0.00002 lr, cosine learning rate schedule.
- 2 epoch 만 학습, 신기하게 CLM 방식으로 학습하되, instruction은 zero-out 함. 그렇게 해서 4096 context를 꽉꽉 채움

# Human Preference Data Collection
- binary comparison protocol
- First wriate a prompts, then choose beetween two sampled model responses, based on provided criteria.
- response 뽑을 때는 서로 다른 모델의 다른 temperature로 뽑음
- We focus on helpfulness and safety.
- Safety data는 둘다 safe, 하나만 safe, 둘다 unsafe 세가지 데이터 bin으로 구성. unsafe 한 샘플이 선택되지 않도록 신경씀
- RM 모델도 weekly로 향상이 되면 progressively 하게 LLAMA2 chat도 향상 시킴.
- 이 역시, iterative 하게 계속 데이터를 생성하는게 중요하구나

# Reward Modeling
- The reward model takes a model response and its corresponding prompt (including contexts from <r>previous turns</r>)
- outputs a scalar score to indicate the quality
- We train two separate reward models, one optimized for helpfulness, and another for safety
- initialize rm from preetrained chat model.(replacing next token prediction head to regression head)
- binary ranking loss
- 우리가 만든거 + open-source preference data랑 합쳐서 사용
  - 우선 오픈소스 데이터로 학습하고, 그 사이 데이터 생성. 그리고 그 뒤에도 쭉 합쳐서 사용( Meta 9 : open 1 비율)
- 이 친구들도 one epoch training
- More data and larger-size model generally improve accuracy.

# Iterative Fine-tuning
- PPO: standard RLHF
- Rejection sampling fine-tuining: K outputs from the model, select the best output for gradient udate. 그니까 제일 좋은 정답들 모아서 다시 파인튜닝
- *Rejection Sampling*: 70B LLAMA2-chat 사용(작은 모델들도 이친구로 만든 데이터 사용). RLHF V3가 만약 v2가 만든 데이터만 사용하면 regression in some capabilities(시쓰기 등에서). 따라서 V1, V2가 만든 데이터도 함께 사용.
- 
