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
