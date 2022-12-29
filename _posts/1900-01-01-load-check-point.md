---
layout: post
date: 2022-12-29 00:00
created_date: 2022-12-29 00:00
title: "[Work] state_dict의 key 이름이 변경 됬을때 로드(Electra)"
description:
comments: true
mathjax: true
category:
- Work
tags:
- NLP
- HuggingFace
---

이런 식으로 수작업 model.decoder.generator_predictions.dense.weight[:] = checkpoint['decoder.lm_head.dense.weight']
<!--more-->

- huggingface 버전이 변경됨에 따라 Electra Architecture의 세부 이름이 바꼈다 예를 들면
- ElectraLMHead -> ElectraGeneratorPredictions 라든가,
- 그럼 이전 버전에서 학습한 모델을 기존 버전에서 로드하면 key 불일치가 발생. 따라서 아래처럼 수작업으로 바꿔줌.
```python
with torch.no_grad():
    model.decoder.generator_predictions.dense.weight[:] = checkpoint['decoder.lm_head.dense.weight']
    model.decoder.generator_predictions.dense.bias[:] = checkpoint['decoder.lm_head.dense.bias']
    
    model.decoder.generator_lm_head.bias[:] = checkpoint['decoder.lm_head.decoder.bias']
    model.decoder.generator_lm_head.weight[:] = checkpoint['decoder.lm_head.decoder.weight']
    
    model.decoder.generator_predictions.LayerNorm.bias[:] = checkpoint['decoder.lm_head.layer_norm.bias']
    model.decoder.generator_predictions.LayerNorm.weight[:] = checkpoint['decoder.lm_head.layer_norm.weight']
    
    model.decoder.generator_lm_head.bias[:] = checkpoint['decoder.lm_head.bias']
'''
