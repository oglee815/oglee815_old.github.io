---
layout: post
date: 2023-08-09 14:08
created_date: 2023-08-09 14:08
title: "[paper] Training language models to follow insstructions with human feedback"
description: Instruction GPT
comments: true
mathjax: true
category:
- Paper
tags:
- OpenAI
- ChatGPT
- InstructGPT
- RLHF
---

ChatGPT의 모태가 되는 Instruct GPT
<!--more-->

<style>
r{color:Red}
o{color:Orange}
g{color:Green}
</style>

너무 유명한 페이퍼라 기억해야 할 몇가지 사실들만 정리

# Summary
- InstructGPT models show <r>improvements in truthfulness</r> over GPT-3 : TruthfulQA benchmark 사용. 2배 정도 성능 향상. 특히 ClosedDomain QA에서 할루시네이션이 적음
- InstructGPT shows <r>small improvements in toxicity</r> over GPT-3, but not bias. : RealToxicityPrompts 사용. 25% fewer toxic output
- <r>Alignment tax</r>, We can <r>minimize performance regressions on public NLP datasets</r> by modifying our RLHF fine-tuning procedure: NLP task에 성능이 떨어지는게 맞긴 맞는듯? anthropic은 근데 그렇지 않다고 했음. 무슨 차이일까? 그리고 이를 줄이기 위해 Pretraining도 함께 진행함(PPO-ptx)
- Our models generalize to the preferences of “held-out” labelers that did not produce any training data: 학습에 동원되지 않은 사람들도 이 모델이 더 다고 평가.
- Public NLP datasets are not reflective of how our language models are used: 사람이 만든 데이터하고, T0, Flan 같은 데이터로 학습한 모델하고 비교하면 후자가 살짝 후짐(1단계 SFT대비). 3단계까지하면 훨씬 InstructGPT가 좋음
