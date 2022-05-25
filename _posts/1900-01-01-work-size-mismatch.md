---
layout: post
date: 2022-05-25 00:00
created_date: 2022-05-25 00:00
title: "[work] layer size mismatch(torch)"
author: oglee
description: layer size mismatch(torch)
comments: true
math: true
category:
- Work
tags:
- pytorch
- huggingface
- form_pretrained
---

huggingface에서 from_pretrained시 이름이 같은 레이어인데 size가 다를때 나는 에러

<!--more-->

- label이 3개인 데이터에 모델을 학습하고 이 모델을 레이블이 10개인 데이터에 적용하기 위해 from_pretrained 하면 에러뜸
- 해결책 예시
```python
from transformers import XLMRobertaForSequenceClassification
from transformers import AutoConfig

config = AutoConfig.from_pretrained("./", num_labels=num_labels, finetuning_task='glue')
model = XLMRobertaForSequenceClassification(config)

state_dict = torch.load("./pytorch_model.bin")
temp = OrderedDict()
for i, j in state_dict.items():   # search all key from model
    name = i.replace("classifier.","")  # change key that doesn't match
    temp[name] = j
model.load_state_dict(temp, strict=False)
```
- https://re-code-cord.tistory.com/entry/Pytorch-Tips-for-Loading-Pre-trained-Model
- https://cchhoo407.tistory.com/37
