---
layout: post
date: 2023-08-05 14:08
created_date: 2023-08-05 14:08
title: "[paper] Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback"
description: Anthropic RLHF paper
comments: true
mathjax: true
category:
- Paper
tags:
- Anthropic
- ChatGPT
- RLHF
---

Anthropic's RLHF
<!--more-->

<style>
r{color:Red}
o{color:Orange}
g{color:Green}
</style>

# Abstract
- We apply preference modeling and <r>reinforcement learning from human feedback (RLHF)</r> to finetune language models to act as helpful and harmless assistants.
- We find this alignment training improves performance on almost <r>all NLP evaluations</r>, and is fully compatible with training for specialized skills such as <r>python coding and summarization.</r>
- We explore an iterated online mode of training, where preference models and RL policies are updated on a <r>weekly -cadence</r> with fresh human feedback data, 
- Finally, we investigate the robustness of RLHF training, and identify a roughly <r>linear relation between the RL reward and the square root of the KL divergence between the policy and its initialization</r>
> 그니까, 학습 된 모델이 초기 모델과 KL이 멀어질수록, RM의 reward가 높아짐

# Introduction
- In this paper we show that we can train a relatively helpful and harmless (HH) natural language assistant by collecting human preference data and applying the techniques of <r>preference modeling (PMing) and reinforcement learning from human feedback (RLHF)</r>
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/b22e6aca-69f4-4b18-89f8-41fb0b9574d0)
- For harmlessness, we invite crowdworkers to adversarially probe or <r>‘red-team’ our language models in order to provoke harmful responses</r>
- RLHF를 하면 대부분의 NLP Tasks에서도 괜찮은 성능을 보인다(그러나 모델 사이즈가 작으면 성능 감소 있음)
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/01e70b3a-79b4-4646-9495-f55a8f4ef1ec)

## Contributions
- Dialogue Preference Datasets : helpfulness and harmlessness data, three tranches of data(one from initial models, one with rejection sampling against early preference models, one with models trained with online RLHF
- Alignment with Human values has many benefits and essentially no cost to performance
  - Smaller models experience severe 'alignment taxes'(13B, 52B에서는 오히려 NLP task도 잘함)
  - 요약 테스크와 관련한 specialized skill을 HH 데이터와 함께 학습하니, 둘다 잘하더라
  - There is a tension between helpfulness and harmlessness
  - one can use OOD(out-of-distribution) Detection techniques to reject most strange and harmful request
- Scaling, RLHF Robustness, and Iterated 'Online' Training
  - We study scaling relations for PM accuracy as a funtion of model and dataset size, and find rougle log-linear trends
  - robustness RLHF, <r>larger PMs are more robust than smaller PMs</r>
  - We find that <r>root(D_KL(Pi||Pi_0)) and reward are approximately linearly related</r> for much of RLHF training.
  - 주마다 새로운 모델을 만들어서 crowdworker와 작업
- Summary of Evaluations
  - NLP and Code Evaluations, Static Alignment Evaluations(HHH Evaluations from BIG-Bench),Bot Adversarial Dialogues and for gender bias, TruthfulQA, BBQ-Lite
  - <r>RLHF improves sentiment towards all groups, but does not remove bias</r>.
  - Human Evluations: Elo scores comparing context-distilled models, base RLHF trained models, final online RLHF models

# Related work
- 우리가 instructGPT와 다른 점은 online learning이다.
  - where we update the models interacting with crowdworkers in order to obtain <r>progressively higher-quality data and fill out the tails of our data distribution</r>
  - specialized skill(coding, summarization

# Data Collection
- User see two possible model responses, and choose one with which to proceed. These two responses may come from the <r>same model, or two different models</r>.
- Croudworker의 2가지 역할
  - 1) 모델에게 질문하기
  - 2) 2개의 답변 중 어느것이 더 helful and honest response 인지 선택(red-teaming은 harmful response 선택)
- Master-qualified US-based MTurk workers

## Helfulness and Harmlessness (Red Teaming) Datasets
- For the <stong>helpfulness</strong> datasets, we asked crowdworkes to have <r>open-ended conversations</r> with our models, asking for help, advice, or for the model to accomplish a task, and <r>to choose the model response that was more helpful.</r>
- For <strong>harmlessness</strong>, we asked crowdworkers to attempt to elicit harmful responses from our models, and to choose the more harmful response offered by the models.
- Note that this means our helpfulness dataset tends to move conversations in a more <r>beneficial direction</r>, while in our red-teaming dataset user responses move conversations in a more <r>harmful direction</r>

## Models deployed to the feedback interface and associated data distribution
- we used 52B LLM with the broad specifications
  - <r>HHH context-distilled 52B Language model</r>: It performs similarly to a plain 52B language model prompted with HHH dialogues
  - <r>Rejection Sampling with a 52B preference model</r>, where samples were generated from a 52B context-distilled LM. In this case the number k of samples was a parameter, but most often we used <r>k=16</r>
  - RLHF Finetuned Models: We used a succession of these models in our interface. We also deployed models trained on different mixtures of helpfulness and harmlessness data.
- Three data distribution
  - A <r>core base dataset</r> collected using only the context-distilled LM. This datset includes <r>44k helpfulness</r> comaprisons and <r>42k red-teaming comparions.</r>
  - <r>RS</r> datasets consisting of 52k helpfulness comparisons and 2k red-teaming comparions using rejection sampling models, where rejection sampling  used a preference model trained on the base dataset.
  - <r>online</r> datasets including data from RLHF models, which were updated on a rougly weekly cadecnce over the course of about five weeks. This dataset contains <r>22k helfulness comparions and no red-teaming data.</r>

## Comparing Models with Elo Scores
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/cf5a5501-3c2c-44ff-8f4b-60ae97e26baf)

# Preference Modeling for Helpfulness and Harmlessness
- 13M ~ 52B 까지 4배씩 증가시키며 테스트
- Preference model pretraining(PMP) : LM Pretraining LR * 01, train on a mixture of comparison data made from StackExchange, Reddit, and Wikipedia.
- We typically only train PMs for <r>a single epoch</r>(fixed learning rate)
- log-linear trends in both dataset and model size
- <r>Preference model scores should predict the probability</r> that humans will prefer one or another model-generated response.
- We observe that PMs trained <r>only on helpfulness data are very well calibrated</r>, but PMs trained on <r>a mixture of helpful and harmless data are slightly under-confident</r>.
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/c71be2e6-efe3-43d2-858c-e7d13577aa50)
- We use <r>PM scores as the reward signal for RL</r>
- 모델들이 High PM score를 받기 시작할 때, return 이 적어질 수 있으므로 online training 이 중요

# Reinforcement Learning from Human Feedback
- 1) Prepare a dataset of comparisons, and <r>train a PM to assign a higher score to the ‘better’ item in
each comparison.</r> In the context of our human feedback experiments, each comparison consists of
a prompt followed by a pair of model-generated responses, with a PM score evaluated at the end of
each response.
- 2). <r>Extract all the prompts from the preceding dataset, and train an RL policy to generate a response
to each prompt autoregressively</r>, with a reward signal provided by the PM score at the end of the
response.
> 그럼 1에서 쓴 데이터를 2에서도 쓴다는 의미?
- 하나의 발화가 timestep, 대화 자체가 trajectory, PM score는 끝에서 주어짐
> 그럼 멀티턴으로 대화하고, 멀티턴에 대해서 하나의 PM Score가 주어진다는 거네? --> 어펜딕스 보니까 아닌듯, 매 발화마다 PM score가 있긴 있는듯
- <r>PMs also become less calibrated at higher scores, so higher rewards do not necessarily imply better performance.</r>
- <r>To stabilize RL training, we use PPO</r>
- ![image](https://github.com/oglee815/oglee815.github.io/assets/18374514/05d5da6a-ee35-4023-b577-2a6d3b3744a6)
- RLHF를 위해서 고픔질의 질문 10개를 Few-shot으로 활용, 추가 질문을 모집. 이때 Large LM을 활용
- 모델이 생성한 질문이나 크라우드 워커가 만든거나 효율이 비슷해서 이 둘을 합침, 137K(사람), 369K 모델 생성

## Robustness Experiments
- PM이 학습 분포와 다른 대화에서는 잘 동작하지 않을 수 있음.
- 이러한 경우, PPO가 잘 말을 해도 PM이 점수를 낮게 줄 수도 있다. 
- 그래서 중간중간 모델의 snapshot을 활용해서 사람들에게 평가하도록 하고 PM score와 비교(true ELO)
- 그러나 위의 방식은 너무 비싸기 때문에, test 용 PM을 따로 만들어서 평가함
- 결국 robustness는 모델이 클수록 좋고 PM score에 높게 도달할 수록 robust하지 않게 됨

## An approximately Linear Replation Between root(D_KL) and Reward

## Tension Between Helpfulness and Harmlessness in RLHF

## Iterated Online RLHF
- We simply train the best RLHF policy we can, and use that to collect comparison data from crowdworkers. Since the policy was trained to optimize for PM score, it should produce responses that are on the upper end of the score distribution.
- We mix the new comparison data with our existing data, and train a new scan of PMs, which we then use to train a new scan of RLHF policies. Then reiterate this process indefinitely.
- 

# Appendix : Preference Modeling
- All PMs go through 3 phase:
  - language model Pretraining
  - Preference model Pretraining: LR은 0.1배 to LM pretraining
  - Human Feedback finetuning : LR 0.01배 to LM pretraining
- <r>마지막에 end-of-context token 붙잉고, 이 토큰으로 PM Score 예측</r>
- lr scheduling이나 warmup 도 중요한듯

