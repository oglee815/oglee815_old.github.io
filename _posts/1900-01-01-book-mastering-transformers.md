---
layout: post
date: 2022-11-23 00:00
created_date: 2022-11-23 00:00
title: "[Book] Mastering Transformers"
author: Savas Yildirim
description: "Mastering Transformers"
comments: true
math: true
category:
- book
tags:
- NLP
- Transfomers
---

Mastering Transformers 
<!--more-->

### Self-attention mechanism
- ![image](https://user-images.githubusercontent.com/18374514/203566128-2471f22b-3f6d-4ea9-a349-b24b084f5f23.png)

### Multi-head attention 
- ![image](https://user-images.githubusercontent.com/18374514/203567247-bc29a6fe-7786-446b-aaf7-e58464549cba.png)
- ![Dp8ERiD](https://user-images.githubusercontent.com/18374514/203567844-ab58e767-da31-4759-8c70-b1a96fec4843.png)

### Simple Distilation code
- ![image](https://user-images.githubusercontent.com/18374514/203574281-7f698ccb-a31b-4c59-a628-46af1a79c925.png) from [link](https://medium.com/huggingface/distilbert-8cf3380435b5)

### How to prune the model
    from torch.nn.utils import prune
    pruner = prune.L1Unstructured(amount=0.2)
    state_dict = distilroberta.state_dict()
    for key in state_dict.keys():
        if "weight" in key:
            state_dict[key] = pruner.prune(state_dict[key])
    distilroberta.load_state_dict(state_dict)

### Quantization
- FP16, Mixed Precision등을 쓰는것
- float 대신 int8로 quantization 하는 법
```
import torch
distilroberta = torch.quantization.quantize_dynamic(
    model = distilroberta,
    qconfig_spec ={torch.nn.Linear : torch.quantization.default_dynamic_qconfig},
    dtye=torch.qint8
    )
```
