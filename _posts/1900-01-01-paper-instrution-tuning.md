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
k { background-color:pink }
</style>

# FLAN: Finetuned language models are zero-shot learners, Google
## Intro
- simple method for <k>improving the zero-shot learning abilities</k> of language model.
- instruction tuning, finetuning language models on a collenction of datasets described via instructions, substantially improves zero-shot performance on unseen tasks
- <k>137B parameter</k> pretrained language model and instruction tune it on over <k>60 NLP datasets verbalized via natural language instruction templates</k>
- <k>surpasses zero-shot 175B GPT3 on 20 of 25 datasets</k>
- <img src='https://user-images.githubusercontent.com/18374514/225216017-f87e0bf9-453e-46a5-9774-3fe85074a2fd.png' width=650>
- GPT가 Few-shot은 잘 했으나 Zero-shot은 그닥이었음.
  - One potential reason is that, without few-shot exemplars, it is harder for models to perform well on prompts hat are <k>not similar to the format of the pretraining data.</k>
- Zero-shot 성능을 높이기 위해 인간의 instruction을 활용해보자! ex. 이거 번역해줘. 이거 긍정일까 부정일까?

## Templates
- 62 text datasets into a single mixture, 12 task clusters
- <img src='https://user-images.githubusercontent.com/18374514/225226122-71ff24d0-eaef-4984-9a93-8bd0e3804658.png' width=650>
- For each datasets, we manually compose <k>ten unique templates</k> that use natural language instructions.
- To increase diversity, we also include up to three templates that <k>"turned the task around,"</k>, (e.g., for sentiment classification we include templates asking to generate a movie review).
- <img src='https://user-images.githubusercontent.com/18374514/225227730-dd0cc691-7e0d-4f7b-aac7-15627972d74d.png' width=650>
  - option을 꼭 주는게 맞을까?
