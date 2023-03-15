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
r { color: Red }
o { color: Orange }
g { color: Green }
</style>


# TODOs:

- <r>TODO:</r> Important thing to do
- <o>TODO:</o> Less important thing to do
- <g>DONE:</g> Breath deeply and improve karma
- 
# FLAN: Finetuned language models are zero-shot learners, Google
- simple method for <mark>improving the zero-shot learning abilities</mark> of language model.
- instruction tuning, finetuning language models on a collenction of datasets described via instructions, substantially improves zero-shot performance on unseen tasks
- **137B parameter** pretrained language model and instruction tune it on over **60 NLP datasets verbalized via natural language instruction templates**
- **surpasses zero-shot 175B GPT3 on 20 of 25 datasets**
- <img src='https://user-images.githubusercontent.com/18374514/225216017-f87e0bf9-453e-46a5-9774-3fe85074a2fd.png' width=650>

