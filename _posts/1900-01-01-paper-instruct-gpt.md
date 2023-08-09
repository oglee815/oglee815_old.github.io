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
- Public NLP datasets are not reflective of how our language models are used: 사람이 만든 데이터하고, T0, Flan 같은 데이터로 학습한 모델하고 비교하면 후자가 살짝 후짐(1단계 SFT대비). 3단계까지하면 훨씬 InstructGPT가 좋음. <r>근데 과연 NLP Task에서는 InstructGPT가 더 좋을까?</r>
- InstructGPT models show promising generalization to instructions <r>outside of the RLHF finetuning distribution</r>: 배우지 않은 테스크도 어느정도 하고, 다른 랭귀지에서도 작동함.
- Stage2, 3는 Iterative 하게 하 수 있음. 이건 Anthropic 하고 똑같음.
- 대부분의 comparison 데이터는 1단계 끝난 SFT, 3단계의 PPO에서 뽑음

# Dataset
- OpenAI API를 통해 Prompts 수집, 인당 200개를 넘지 않도록 함
- validation과 test 데이터에는 반드시 학습 데이터를 만든 사람의 ID가 포함되지 않도록 해서 공정한 평가가 이루어지도록 함.
- 레이블러에게 3가지 형태로 명령을 줌
  - Plain : arbitrary task, sufficient diversity
  - Few-shot : multiple query/response pairs for the instruction
  - User-based : use case 를 주고 여기에 맞는 데이터 생성
- Stage1: 13K Training, Stage2: 33K, Stage3: 31K(only from API)
- Illustrative Prompts를 레이블러한테 보여주기도 함
- 명확한 자연어 명령, Fewshot, implicit contiuation(개구리관련 스토리를 알려주고 이어가게 한다든가)
- Task가 intention이 헷갈리면 Skip
- <r>사용자가 harmful 한 요청을 했을때도 일단은 helpfulness를 우선으로 함</r>
- 트레이닝에 전혀 관여하지 않은 사람들로 평가를 함(그러나 레이블러들하고 평가하는게 크게 다르지 않았음)

# Models
- <strong>SFT</strong> : <r>16epochs</r>, cosine lr decay, residual dropout 0.2. RM 모델로 베스트 모델을 선택 1에폭만 지나도 오버피팅 되긴하는데 그래도 human preference는 상승하는 걸로 보였음
- RM: SFT에서 final layer만 걷어내고 학습함. 6B Rm 사용. input은 prompt와 response, output은 scalar reward.
> appendix에는 여러 NLP task에 파인튜닝 한 모델이라고 되어있음.
- reward의 차이는 사람이 더 해당 response를 선호할 log odd
- 하나의 prompt에 대해서 K개의 아웃풋 생성, kC2 만큼의 경우의 수 만듦. 이걸 다 그냥 학습 데이터로 쓰면 오버피팅이 심하기 때문에, 하나의 배치에서 학습하도록 처리. 그럼 그냥 forward도 k번 하면 되니까 훨씬 간편하고, 오버피팅도 안됨
> 하나의 배치에 특정 샘플들을 모으는거 어케하누? 그리고 우리는 그게 안되면 그냥 k=2로 해버리면 되긴함
> 다시 읽어보니 하나의 배치에는 무조건 하나의 prompt에 대한 response들로만 꽉차 있는것.!!! 그래서 로스도 kC2 로 나눠준다.
> 학습 돌릴때, 데이터를 RandomSampling해서 배치 구성하는게 아니고 순서대로 그냥 뽑아내도록 구현하면 되려나?
- RM loss is invariant to shifts in reward, we normalize the rm using a bias so that the labeler demonstrations achieve a mean score of 0 before doing RL.
> RL을 하기 전에 demonstration 데이터가 0점을 받도록 했다는 건가?, invariant to shift in reward 라는건 loss가 reward 분포가 바껴도 상관 없다는거네.
- *RL* : Per-token KL Penalty from the SFT model to mitigate over-optimization of the reward. *Value function is initialized from the RM* we call these models PPO
- GPT-3 on FLAN, T0는 1M 샘플로 학습함

# Evaluation
- InstructGPT를 위해 사용자가 제출한 API만 사용하면 GPT3에게 불리하기 때문에 GPT3 API를 통해서 수집한 데이터도 평가에 활용
- RealToxicityPrompts는 우리도 번역해서 사용해야 겟음
- <r>1.3B PPO가 175B SFT보다도 살짝 좋다</r>
- Hallucination은 확실히 SFT가 PPO보다 차라리 낫다

# Limitations
- Most comparions are only labeled by 1 contractor for cost reasons.
- Comparions are also not necessarily the most efficienet way of providing an alignment signal
- For example, we coud have labelers edit model responses to make them better, or generate critiques of model reponses in natural language. 

# Appendix
- 하나의 prompt 당 K output을 뽑았기 때문에, 실제로는 prompt 보다 몇배 이상 학습을 시킨거임
- 미친 175B는 배치가 8이네?
- RM 모델 조차 6B GPT-3를 다양한 NLP task에 finetuning 시킨 모델임
