---
layout: post
date: 2022-04-13 00:00
created_date: 2022-04-13 00:00
title: "[work] vim에서 default로 mouse=r 설정"
author: oglee
description: vim에서 default로 mouse=r 설정
comments: true
math: true
category:
- Work
tags:
- Vim
- Docker
---

huggingface에서 from_pretrained시 이름이 같은 레이어인데 size가 다를때 나는 에러

<!--more-->


- from transformers import XLMRobertaForSequenceClassification
from transformers import AutoConfig

config = AutoConfig.from_pretrained("./", num_labels=num_labels, finetuning_task='glue')
model = XLMRobertaForSequenceClassification(config)

state_dict = torch.load("./pytorch_model.bin")
temp = OrderedDict()
for i, j in state_dict.items():   # search all key from model
    name = i.replace("classifier.","")  # change key that doesn't match
    temp[name] = j
model.load_state_dict(temp, strict=False)
- https://cchhoo407.tistory.com/37
