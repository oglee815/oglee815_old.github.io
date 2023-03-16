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
- `option을 꼭 주는게 맞을까?`
- Option을 주는게 당연히 성능 향상에는 도움이 되지만, 실제 사용 할 때 매번 option을 넣기엔 애바인데?

## Model
- <k>Lambda-PT</k>, a dense left-to-right, decoder-only transformer language model of 137B parameters
- pretrained on a collection of <k>web</k> documents(including thoese with computer <k>code</k>), <k>dialog data and wikipedia</k>, tokenized into 2.49T BPE tokens with a 32k vocabulary using the sentencepiece.
- <k>10% of data was non-english</k>.
- PT가 붙은 이유는 dialog에 fine-tune 되지 않은 lambda이기 대문

## Instruction tuning procedure
- mix all datasets and randomly samples from each dataset.
- To balance the different sizes of datasets, we limit the number of training examples per dataset to <k>30k</k> and follow the examples-proportional mixing scheme with a mixing rate maximum of <k>3k</k>
- <k>30K</k> gradient steps with <k>8192 batch size</k> using <k>Adafactor Optimizer(3e-5)</k>
- sequence length for input:1024 output:256 
- `packing이 뭘까?`
- 60 hours on a TPUv3 with 128 cores.

## Results
- best dev template?
- NLI : GPT보다 훨씬 성능이 좋음. pretraining 단계에서 NLI 예시들이 거의 나타나지 않았기 때문.
  - template : "does < premise > mean that < hypothesis >?"
   
## How performance is affected by the number of clusters and tasks used in instruction tuning?
- sentiment analysis 빼고 task를 늘릴 수록 unseen task에서 성능이 늘어남.

## Scaling laws
- 422M, 2B, 8B, 68B, 137B
- <k>8B 까지는 오히려 성능이 떨어짐</k>
- One potential explanation for this result could be that for small-scale models, learning the ~40k tasks used during <k>instruction tuning fills the entire model capacity</k>, causing these models to perform worse on new tasks.

## Role of Instruction
- unseen task를 잘하는게 Multi task learning의 효과인지 보기 위해서, 
  - 1) instruction 없이 그냥 학습, 평가 때는 instruction을 주긴 줌(왜냐면 이것도 안주면 아예 맞출 가능성이 없기 때문)
  - 2) dataset 이름만 주기 ex) Translation:WMT'14 to French
  - 근데 데이터셋 이름을 평가때도 주면.. 이건 치팅인데?
- <img src='https://user-images.githubusercontent.com/18374514/225252837-c70afb65-fe99-4e6c-ab5d-ccf503d5a77f.png' width=300>

## Instruction with few-shot examplars
- Few-shot을 위한 몇개의 샘플(instruction포함) + 새로운 입력 with instruct 하면 성능이 훨씬 좋아짐
- Few-shot을 위해 최대 16개 샘플을 넣었음.

## Instruction tuinng faciliates prompt tuning
- FLAN + Continuous Prompt Tuning.
- <k>제일 신기한 부분.. FLAN하고 continuous prompt를 섞으면 성능이 10점 이상 좋아진다</k>
- <k>SuperGLUE에 대해서, N-1개로 insturction tuning 한 다음, 나머지 1개에 대해서만 continous prompt 튜닝을 추가적으로 하고 그 해당 데이터셋에 대하여 평가한 것 같은? </k> 확실하진 않음.
- 

## Datasets per task cluster & templates per datset
- <img src='https://user-images.githubusercontent.com/18374514/225262595-5e24496b-53bb-4ad3-95ca-2c9c4639aa1c.png' width=400>
- 클러스터 별로 dataset 사이즈가 중요한듯. 오히려 템플릿은 크게 중요하지가 않네. 

# 2210 Scaling instruction-finetuned language models
## Intro
- In this paper we explore instruction finetuning with a particular focus on <k>(1) scaling the number of tasks, (2) scaling the model size, and (3) finetuning on chain-of-thought data.</k>
- We find that instruction finetuning with the above aspects <k>dramatically improves performance</k> on a variety of model classes (PaLM, T5, U-PaLM), prompting setups (zero-shot, few-shot, CoT), and evaluation benchmarks (MMLU, BBH, TyDiQA, MGSM, open-ended generation, RealToxicityPrompts).
- <img src='https://user-images.githubusercontent.com/18374514/225526761-c913b519-e74a-435a-b528-a953f1964bf3.png' width=500>
- <img src='https://user-images.githubusercontent.com/18374514/225526981-cc2f3898-9f78-4013-9d5f-977f2f952c7a.png' width=500>
- 9개 데이터셋에 대하여 CoT를 <k>수작업</k>으로 만들었음
- <img src='https://user-images.githubusercontent.com/18374514/225543110-bc1cae8a-c089-415f-a508-969512e8f5ae.png' width=500>
- 위 기준에서는 무조건 task가 많으면 많을수록 좋다. 단 8B 이상에서..
- 282까지는 매우 많이 상승하고, 그 뒤는 조금씩...
- 그 이유는 282 이후에는 diverse가 실제로 적은 task들. 새로 배우는게 없음.
- Another explanation is that most of the gains from multi-task instruction finetuning come from the model learning to better express knowledge that it <k>already knows from pretraining, and more than 282 tasks does not help too much</k>. This second explanation could make sense since the pre-training data consists of 780B tokens, while instruction finetuning only uses 1.4B tokens

# 2023 The Flan Collection: Designing Data and Methods for effective instruction tuning
## Intro
- We find task <k>balancing and enrichment techniques</k> are overlooked but critical to effective instruction tuning, and in particular, training with <k>mixed prompt settings (zero-shot, few-shot, and chain-of-thought)</k> actually yields stronger (2%+) performance in all settings.
- we show Flan-T5 requires <k>less finetuning to converge higher and faster than T5 on single downstream tasks</k>—motivating instruction-tuned models as more computationally efficient starting checkpoints for new tasks.
- we evaluate the methods and results of open sourced instruction generalization efforts, comparing their finetuning techniques and methods.
