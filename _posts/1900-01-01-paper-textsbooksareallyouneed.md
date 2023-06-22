---
layout: post
date: 2023-06-21 14:08
created_date: 2023-06-21 14:08
title: "[Paper] Textbooks are all you need"
description: Textbookas are all you need
comments: true
mathjax: true
category:
- Paper
tags:
- NLP
- Code Language Model
---

We introduce phi-1, a new large language model for code, with significantly smaller size than competing models
<!--more-->

<mark style='background-color:pink'>
< style >
r { color: Red }
o { color: Orange }
g { color: Green }
</style>

# Abstract
- We introduce phi-1, a new large language model for code, with <r>significantly smaller size</r> than competing models
- <r>phi-1 is a Transformer-based model with 1.3B parameters, trained for 4 days on 8 A100s, using a selection of “textbook quality” data from the web (6B tokens)</r> and synthetically generated textbooks and exercises with GPT-3.5 (1B tokens). 
- Despite this small scale, phi-1 attains pass@1 accuracy 50.6% on HumanEval and 55.5% on MBPP. It also displays surprising emergent properties compared to phi-1-base, our model before our finetuning stage on a dataset of coding exercises, and phi-1-small, a smaller model with 350M parameters trained with the same pipeline as phi-1 that still achieves 45% on HumanEval.
