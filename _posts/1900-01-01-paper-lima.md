---
layout: post
date: 2023-05-28 14:08
created_date: 2023-05-28 14:08
title: "[paper] Less is more(LIMA)"
description: LIMA paper
comments: true
mathjax: true
category:
- Paper
tags:
- LIMA
- ChatGPT
---

1000여개의 샘플로도 Instruction tuning 가능

<!--more-->

<!-- <mark style='background-color:pink'> -->
<style>
r{color:Red}
o{color:Orange}
g{color:Green}
</style>

# Abstract
- Large language models are trained in two stages: 
  - (1) unsupervised pretraining from raw text, to learn general-purpose representations, and 
  - (2) large scale instruction tuning and reinforcement learning, to better align to end tasks and user preferences.
- We measure the relative importance of these two stages by training LIMA, a 65B parameter LLaMa language model fine-tuned with the standard <r>supervised loss on only 1,000 carefully curated prompts and responses</r>, <g>without any reinforcement learning or human preference modeling.</g>
- LIMA demonstrates remarkably strong performance, learning to follow specific response formats from only a handful of
examples in the training data, <r>including complex queries that range from planning trip itineraries to speculating about alternate history.</r>
- Moreover, the model tends to <r>generalize well to unseen tasks that did not appear in the training data.</r>
- In a controlled human study, responses from LIMA are either equivalent or strictly preferred to GPT-4 in 43% of cases; this statistic is as high as 58% when compared to Bard and 65% versus DaVinci003, which was trained with human feedback.
Taken together, these results strongly suggest that <r>almost all knowledge in large language models is learned during pretraining</r>, and only <r>limited instruction tuning data is necessary to teach models to produce high quality output</r>

# Introduction
- We hypothesize that alignment can be a simple process where the model learns the style or format for interacting with users, to expose the knowledgeand capabilities that were already acquired during pretraining.
- To test this hypothesis, we curate 1,000 examples that approximate <r>real user prompts and high-quality responses</r>. We select <r>750 top questions and answers from community forums</r>, such as Stack Exchange and wikiHow, sampling for quality and diversity.
- In addition, <r>we manually write 250 examples of prompts and responses</r>, while optimizing for <r>task diversity and emphasizing a uniform response style in the spirit of an AI assistant.</r>
- Finally, we train LIMA, a pretrained 65B-parameter LLaMa model [Touvron et al., 2023] fine-tuned on this set of 1,000 demonstrations.
- 나머진 대충 성능 좋았다는 얘기

# Alignment Data
- We define <r>the Superficial Alignment Hypothesis</r>: A model’s knowledge and capabilities are learnt <r>almost entirely during pretraining</r>, while <r>alignment teaches it which subdistribution of formats should be used when interacting with users</r>
- To that end, we collect a dataset of 1,000 prompts and responses, where the outputs (responses) are <r>stylistically aligned with each other, but the inputs (prompts) are diverse.</r>
- Specifically, <r>we seek outputs in the style of a helpful AI assistant</r>
- We curate such examples from a variety of sources, primarily split into <r>community Q&A forums and manually authored examples.</r> We also collect a test set of <r>300 prompts and a development set of 50</r>.
- 최대한 다양한 도메인으로, 하이 퀄러티, I, my, link, image 등을 필터링
- We also include <r>13 training prompts with some degree of toxicity or malevolence</r>. We carefully write response that partially or fully reject the command.
- <r>50 Supernatural language generation tasks</r> such as summarization, paraphrasing, style transfer, and pick <r>a single random example</r> from each one. --> 그리고 스타일을 맞추기위해 조금 고침

# Training
- 65B LLaMa
- EOT token at the end of each utterance.
- 15 epoch, AdamW, 1e-5~1e-6. batch 32, 2048 msl
- One notable deviation from the norm is the use of residual dropout
- 5~10 에폭 사이에서 가장 좋은 모델 선택

# Why is Less More
- Diversity: more diverse Stack Exchange data yields significantly higher performance.
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/c34cc34b-f651-4e7f-b83b-c8f1bc63fa3f)
- 데이터 양은 더블링을 해도 나아지지 않음(7B에 대해서)

# Multi-Turn Dialogue
- Multi turn으로 학습 안해도 6/10 이 성공. 보통 3개 턴에서 실패
- 30개의 멀티턴 대화를 수집함. 10개는 직접 만듦, 20개는 Stack Exchange에서 수집 
- 싱글턴 1000개 + 멀티턴 30개로 학습

# Discussion
- Primarily, the <r>mental effort in constructing such examples is significant and difficult to scale up.</r> 
- Secondly, LIMA is <r>not as robust as product-grade models</r>; while LIMA typically generates good responses, an unlucky sample during decoding or an adversarial prompt can often lead to a weak response
