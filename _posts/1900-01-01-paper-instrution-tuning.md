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

  
