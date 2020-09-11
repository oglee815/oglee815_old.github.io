---
layout: post
date: 2020-09-11 11:42
title: PowerBERT
description: Accelerating BERT Inference via Progressive Word-vector Elimination
comments: false
category: 
- NLP
tags:
- NLP
- Distilation
---

## 어떤 내용의 논문인가요? 👋

상대적으로 덜 중요한 Word vector를 제거함으로써, GLUE Task 에서 BERT 대비 Acc의 손실은 1% 이하로 유지하면서 속도는 최대 4.5배 향상함.

<!--more-->

## Abstract (요약) 🕵🏻‍♂️

> We develop a novel method, called PoWER-BERT, for improving the inference time of the popular BERT model, while maintaining the accuracy. It works by: <strong>a) exploiting redundancy pertaining to word-vectors (intermediate encoder outputs) and eliminating the redundant vectors. b) determining which word-vectors to eliminate by developing a strategy for measuring their signiﬁcance, based on the self-attention mechanism. c) learning how many word-vectors to eliminate by augmenting the BERT model and the loss function.</strong> Experiments on the standard GLUE benchmark shows that PoWER-BERT achieves up to 4.5x reductionin inference time over BERT with < 1% loss in accuracy. <strong>We show that PoWER-BERT offers signiﬁcantly better trade-off between accuracy and inference time compared to prior methods</strong>. We demonstrate that our method attains up to 6.8x reduction in inference time with < 1% loss in accuracy when applied over ALBERT, a highly compressed version of BERT. The code for PoWER-BERT is publicly available at https://github.com/IBM/PoWER-BERT.


## 이 논문을 읽어서 무엇을 배울 수 있는지 알려주세요! 🤔

### 1. Introduction

#### Motivation

- BERT 가 NLP 분야에서 significant success 하다는 건 두말하면 입아픔
- 하지만 실제 서비스에서는 성능 만큼이나 resource 와 latency 문제도 중요함.
- 따라서 BERT의 computational demands를 줄이기 위한 연구를 진행함

#### Previous Work
- **ALBERT** : Parameter Sharing 과 Decomposing Embedding Layer 방법으로 Model Size를 획기적으로 줄였으나, Inference Time에서 이득은 없음(계산량 자체는 줄이지 못했으므로).
- **Knowledge Distillation (DistilBERT, BERT-PKD)** 이나 , **Encoder/Head-Prune** 방식은 Accuracy 에서 상대적으로 심각한 저하를 가져옴.

#### Our Objective and Approach
- Object : Accuracy는 어느정도 유지하면서 Inference Time은 대폭 줄이는 것이 목적
- Approach : **모델 사이즈는 유지. 대신 Encoder 를 거치는 동안 생성되는 Word Vector중 일부를 제거함.**
- Main Contributions
    - 불필요한 Word Vector를 제거하는 Scheme 인 **PoWER-BERT 제안(Progressive Word-vector Elimination for inference time Reduction of BERT)**
    - GLUE Tasks 에서 BERT보다 4.5x 빠르고 성능 하락도 1% 이하로 유지
    - 다른 Reduction Methods 와 Speed & Acc 사이의 Trade off 비교
    - ALBERT에도 적용하면 6.8x 빨라짐.
- 기타 논문에서 언급한 Reduction Methods : 
    - pruning network connections, pruning filters/channels from CNN, weight quantization, Knowledge Distillation, singular value decomposition of weight matrices
- 강조하는 특징 : PoWER-BERT는 모델의 파라미터 자체는 그대로 유지하되, word-vectors를 제거하는 방식이므로 기존의 BERT, ALBERT 등의 구조에 바로 적용이 가능하다는 장점이 있음.

### 2. Background

- BERT Base size : 12 encoders, 12 self-attention heads, hidden size 768
- [BERT architecture](https://peltarion.com/knowledge-center/documentation/modeling-view/build-an-ai-model/blocks/bert-encoder?fbclid=IwAR0YUclKFzwx-2MEqRb_X_yTePqFju2E_oLHbcmrnWTmTIxoYe5gnRgH6CE)
- Self-Attention : 
![self-attention](https://user-images.githubusercontent.com/18374514/90849067-5f88cb00-e3a9-11ea-9cdf-a8233a69159e.png)
![self-attention2](https://user-images.githubusercontent.com/18374514/90849828-2a7d7800-e3ab-11ea-981e-7aeee72fa8a9.png)
M : N X 768 input, W<sub>q</sub><sup>h</sup>, W<sub>k</sub><sup>h</sup>, W<sub>v</sub><sup>h</sup> : query, key, value matrix


### 3. PoWER-BERT Scheme

#### Motivation
- BERT로 Classification 문제를 해결 할 때, 사용되는 토큰은 ***[CLS]*** 뿐임.
- 그러나 CLS 외에 다른 위치의 토큰을 Classificaiotn 에 사용했을 때도 크게 Acc 가 달라지지 않았음. (평균적으로 1.2% 차이)
- 따라서, Word Vector 간에는 **diffusion(흩어짐) of information** 이 존재함을 발견함.

#### Diffusion of Information
- Word Vector 가 Encoder 를 통과하면서 점차적으로 비슷한 정보를 담게 됨(due to the self-attention)
![cosine_similarity](https://user-images.githubusercontent.com/18374514/90853529-6289b880-e3b5-11ea-9d89-d4b9bdc00677.png)
SST-2 데이터 셋에 대한 Word Vector 간 Cosine Similarity. 매 레이어별로 <sub>128</sub>C<sub>2</sub>의 모든 조합에 대한 Average 값.
- 위 그림에서 처럼, Word-Vector 끼리 비슷한 정보를 담고 있게 되므로, ***[CLS]*** 외에 다른 Vector를 사용해도 성능이 큰 차이가 없음.
- PoWER-BERT에서는 마지막 레이어에서만 Word Vector 중 일부를 제거하는게 아니라, **전체 Encoder Pipeline 에 따라서 점차적으로 제거하는 것이 핵심**

#### PoWER-BERT Components
- 2가지 Components 가 존재.
    - a retention configuration : 각 레이어 별로 몇 개의 Word Vector만 가져갈지에 대한 Configuration. 아래 그림의 경우는 
(80,73,70,50,50,40,33,27,20,15,13,3)
![retention](https://user-images.githubusercontent.com/18374514/90855147-cf06b680-e3b9-11ea-9418-58e975d0dddd.png)
    - Word Vector Selection : 어떤 word-vectors를 가져가야 하나?

#### Word-vector Selection
- Static and Dynamic Strategies
    - Static : 매 레이어 별로 지워야 할 개수와 위치를 고정함. 
        - Head-WS : 순서대로 앞에 J개만 선택
        - Rand-WS : 랜덤하게 J개만 선택
    - Dynamic : 레이어나 Dataset 에 따라서 다르게 선택함.
        - Attn-WS : Attention Mechanism 을 이용해서 

- Attention-based Scoring
![significant_score](https://user-images.githubusercontent.com/18374514/90867090-cfaa4780-e3cf-11ea-8940-3dcbbc8bc1da.png)

- Word-vector Extraction 
    - self-attention 과 feed-forward network 사이에 extract layer를 추가하여 Score 기준 Top L개를 선택함. 아래 그림에서는 4개, 2개 선택.
![extract_layer](https://user-images.githubusercontent.com/18374514/90868238-8a871500-e3d1-11ea-9e8b-f1265841dc75.png)

- Validation of the Scoring Function
    - Scoring function 의 유효성 검증을 위해 **Mutual Information(MI)** 이용하여 실험 진행.
        - X : BERT의 아웃풋, Y : PoWER-BERT의 아웃풋. MI(X|Y) 는 높을 수록 좋음.
    - 실험 결과, Score 가 높은 단어를 제거 할 수록 MI가 감소함.
    - 즉, Score 와 final classification 에 대한 영향도는 양의 상관관계에 있다.
![mutual](https://user-images.githubusercontent.com/18374514/90885415-84eaf880-e3ec-11ea-94d2-6aafb6219d1f.png)
    - K가 증가할수록(스코어가 높아질수록), MI는 커진다. 즉, Score가 높은 Word를 제거하면 MI가 감소할 가능성이 있다. 반대로 Score가 낮은 Word를 제거하면 MI는 증가. (K : j번째 인코더의 word vector들 중, Score가 높은 순위)
    - Word가 제거되는 Encoder 가 깊어질수록 MI가 증가 -> Score가 높은 단어는 더 깊은 레어에서 제거되야 한다. 처음부터 중요한 단어를 제거해버리면 MI가 낮아짐.

#### Retention configuration
- 각 레이어 별로 몇 개의 word vector를 남겨야 하는가? --> Score기준으로 sorting 하여 결정
- Soft-extract layer : extract layer는 Score를 기준으로 word-vector를 남길지 버릴지 결정하지만, soft-extract는 모든 vector를 남기되 각 score에 따라서 남기는 정도를 조절.
- r<sub>j</sub>[N] 는 0-1사이 값을 갖는 학습 가능한 파라미터(retention parameter). 해당 위치의 vector를 얼마만큼 남길지 결정.
![soft-extractor](https://user-images.githubusercontent.com/18374514/90894715-0007db00-e3fc-11ea-8cbd-715a90a16909.png)

#### Loss Function
- mass : j번째 Encoder의 score 순으로 sorting 된 vector들에 적용되는 r 값의 합 -> 이걸 최소화 하도록 Loss 구현
- theta : BERT 의 파라미터, L : Cross entropy Loss, lambda : hyper-param
- r 값은 1로 초기화 됨-> 즉 처음에는 모든 vector 가 순수하게 전달됨.
![loss](https://user-images.githubusercontent.com/18374514/90907702-45cd9f00-e40e-11ea-8dc4-d5bb26513962.png)
![mass](https://user-images.githubusercontent.com/18374514/90907899-99d88380-e40e-11ea-9c84-b37e3f051a24.png)

#### Training PoWER-BERT
총 3개의 스텝으로 진행
1) ***Fine-tunning*** : Pre-trained BERT 모델로 Fine-tuning
2) ***Configuration-search*** : Soft-extract Layer를 fined-tunned 모델에 추가하고 Loss function을 수정. Inference Time과 acc 사이의 적절한 trade-off를 찾기 위해 Lambda 와 retention parameter 를 학습함. 이때 LR은 10~100배까지 크게 사용.
3) ***Re-training*** : Soft-extract layer를 extract layer로 대체. 제거될 word-vector들의 숫자는 2)에서 찾은 configuration으로 결정. 어떤 word vector를 제거할지는 significance score로 결정. **(CLS는 제거되지 않음)**

### 4. Evaluation

![table2](https://user-images.githubusercontent.com/18374514/90912405-8250c900-e415-11ea-9517-70ad48fb8676.png)
- BERT 대비
    - lambda를 조절함으로써 Acc Loss 는 1% 밑으로 유지하면서 Inference time에서 2.0x ~ 4.5x 빠름. 
    - Input Sequence Length 가 256인 Task의 경우, BERT는 12 X 256 = 3072 개의 Word-Vector 사용. 
    - PoWER-BERT의 경우, 다음의 configuration 사용하면 (153,125,111,105,85,80,72,48,35,27,22,5) 총 868개만 필요.
    - Self-Attention 과 feed-forward network 모듈의 연산량은 Word Vector개수에 비례하므로, 속도가 빨라짐,

- 다른 Methods 와 비교
![method_comparision](https://user-images.githubusercontent.com/18374514/90913718-9e556a00-e417-11ea-9a61-863045c65985.png)

- ALBERT에 적용
    - 마찬가지로 Acc Loss는 1%미만으로 유지하고 속도는 2~6.8x 빨라짐.
> 왜 QQP에서 유독 속도 향상이 클까? 추측으로는 QQP 데이터셋에의 example들의 대부분이 짧은 문장이라서. 
![Table3](https://user-images.githubusercontent.com/18374514/90912492-a7ddd280-e415-11ea-8d0c-a78e24a42ee7.png)

- Ablation Test : Static vs Dynamic Word Selection
![ablation](https://user-images.githubusercontent.com/18374514/90915198-16bd2a80-e41a-11ea-91f6-05200f141b33.png)

- Anecdotal Examples
    - SST-2 (감정분석) 예제에 대한 Word Vector Elimination 예제. Retention Config (7,7,7,7,4,4,4,4,2,2,2,2)
    - 두 예제 모두 첫번째는 stop words와 punctuation이 제거됨.
    - 최종 레이어에서 선택되는 두 개의 단어들로도 충분히 감정 분석이 가능(Diffusion of information 덕분)
![anecdotal](https://user-images.githubusercontent.com/18374514/90915467-92b77280-e41a-11ea-8746-f1f4912d6f02.png)

### 5. Conclusion
- PoWER-BERT는 덜 중요한 Word vector를 제거함으로써 GLUE Task 에서 BERT 대비 Acc의 손실은 1% 이하로 유지하면서 속도는 4.5배 향상함.
- 다른 Method에 비해서 Acc와 Speed 간의 Trade-off를 매우 향상시킴.
- 또한, ALBERT에도 적용가능.
- 앞으로 번역이나 요약과 같은 Task에도 적용할 예정.

## 같이 읽어보면 좋을 만한 글이나 이슈가 있을까요?

경량화를 위한 다른 방법들 : <a>https://blog.est.ai/2020/03/%EB%94%A5%EB%9F%AC%EB%8B%9D-%EB%AA%A8%EB%8D%B8-%EC%95%95%EC%B6%95-%EB%B0%A9%EB%B2%95%EB%A1%A0%EA%B3%BC-bert-%EC%95%95%EC%B6%95/</a>

하이퍼 파라미터 : <a>https://proceedings.icml.cc/static/paper_files/icml/2020/6722-Supplemental.pdf</a>

## 레퍼런스의 URL을 알려주세요! 🔗

논문 : <a>https://proceedings.icml.cc/static/paper_files/icml/2020/6722-Paper.pdf</a>
Github : <a>https://github.com/IBM/PoWER-BERT</a>
BERT Architecture : <a>https://peltarion.com/knowledge-center/documentation/modeling-view/build-an-ai-model/blocks/bert-encoder?fbclid=IwAR0YUclKFzwx-2MEqRb_X_yTePqFju2E_oLHbcmrnWTmTIxoYe5gnRgH6CE</a>
