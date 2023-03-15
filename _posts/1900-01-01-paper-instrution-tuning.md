---
layout: post
date: 2023-03-15 00:00
created_date: 2023-03-15 00:00
title: "[paper] Instruction Fine-tuning 관련 논문들"
description:
comments: true
mathjax: true
category:
- Paper
tags:
- LLM
- Instruction-finetuning
---

FLAN, T0 등등

<!--more-->

<style>
k { background-color:lemonchiffon }
</style>

# FLAN: Finetuned language models are zero-shot learners, Google
- simple method for <k>improving the zero-shot learning abilities</k> of language model.
- instruction tuning, finetuning language models on a collenction of datasets described via instructions, substantially improves zero-shot performance on unseen tasks
- <k>137B parameter</k> pretrained language model and instruction tune it on over <k>60 NLP datasets verbalized via natural language instruction templates</k>
- <k>surpasses zero-shot 175B GPT3 on 20 of 25 datasets</k>
- <img src='https://user-images.githubusercontent.com/18374514/225216017-f87e0bf9-453e-46a5-9774-3fe85074a2fd.png' width=650>
- GPT가 Few-shot은 잘 했으나 Zero-shot은 그닥이었음.
  - One potential reason is that, without few-shot exemplars, it is harder for models to perform well on prompts hat are <k>not similar to the format of the pretraining data.</k>
