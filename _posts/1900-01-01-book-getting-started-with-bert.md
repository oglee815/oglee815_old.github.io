---
layout: post
date: 2022-11-20 00:00
created_date: 2022-09-08 00:00
title: "[Book] Getting Started with Google BERT"
author: oglee
description: "Getting Started with Google BERT"
comments: true
mathjax: true
category:
- book
tags:
- NLP
- BERT
---

Getting Started with Google BERT by Sudharsan Ravichandiran
<!--more-->

# Test
- LR= 2 $$e^{-3}$$ * min( $$step^{-0.5}$$, $$step * warmup^{-1.5}$$), warmup=10000
- LR= 2 $e^{-3}$
- 
### Positional Encoding에 관한 설명
- pos = the position of the word in a sentence, i = the position of the embedding
- ![image](https://user-images.githubusercontent.com/18374514/188919880-c7da1047-f3fa-48c6-b3e1-e98ff17624a0.png)
- <img src="https://user-images.githubusercontent.com/18374514/188919806-0d726905-a441-489a-8c4e-867b8952b893.png" width="300">

### Whole word masking에서 주의할 점
- whole word를 마스킹하다가 만약 masking ratio가 15%를 넘어가게 되면 다른 토큰을 원복시킬 수도 있다.

### BERT의 pretraining 시 하이퍼파람
- 배치 256, 100M steps, lr=1e-4, B1=0.9, B2=0.999, warmup=10K
- 0.1 dropout, GELU Activation.
- <img src="https://user-images.githubusercontent.com/18374514/188924033-29df04ec-918a-4808-a7ec-cc243bfc6956.png" width="300">

### Byte Pair Encoding vs Byte-level byte pair encoding vs WordPiece
- Byte-pair Encoding 
  - 만약 우리 corpus에 다음과 같은 단어들이 있다고 가정하자, cost는 등장 횟수
  - <img src='https://user-images.githubusercontent.com/18374514/189703095-f8f9d1de-4ebc-450c-a892-43e4919ebfd8.png' width='300'>
  - 그리고 우리 vocab size를 14개로 정했다고 가정.
  - 1) 모든 unique character를 vocab에 추가
    - <img src='https://user-images.githubusercontent.com/18374514/189703595-eec2197d-0294-4d22-b2dc-3f49c675bb9f.png' width='300'>
  - 2) 그럼, vocab 이 11개가 되므로, 3개가 부족함. 3개를 더 추가해야함.
  - 3) 등장 횟수 순서로 보면, st와 me가 각각 4번씩 등장하므로, vocab에 추가함    
  - 4) 그 다음에, me+n이 2번 등장하므로 men 도 추가
    - <img src='https://user-images.githubusercontent.com/18374514/189704532-9be2b8c5-0474-43c9-b320-9d0787c115ea.png' width='300'>
  - 5) 그럼 최종적으로 아래처럼 됨.
    - <img src='https://user-images.githubusercontent.com/18374514/189704758-d891f635-5be4-43bf-99bc-67e65a18fbb6.png' width='300'>

- Byte-level byte pair Encoding(BBPE)
  - Character를 Byte로 변경함, 그 후 BPE 실행
  - ex) best -> 62 65 73 74
  - Multilingual settting에서 장점이 크다.

- WordPiece
  - BPE와 기본적으로 비슷하나, frequency 기반으로 merge하지 않고 likelihood에 기반함
  - train set을 대상으로 language model를 우선 학습해서(통계적), maximum likelihood를 갖는 char 조합을 merge함.

### RoBERTa가 BERT와 다른 점.
- Dynamic Masking,  Removing NSP task, More Data Point(CC-News), large batch(256 -> 8K), Shorter training(1M -> 500K), BBPE as a tokenizer(50K vocab)

### SpanBERT
- Span boundary objective(SBO) 
  - <img src='https://user-images.githubusercontent.com/18374514/190318897-6ae1775c-174f-4e4e-8826-eb351a9db1bb.png' width='500'>
  - 위 그림에서 x7을 예측할 때, boundary의 시작과 끝인 R5와 R10만을 활용함.
  - 근데 X6나 x8 등을 예측할 때도 동일하게 R5와 R10을 활용하기 때문에, 어떤 토큰을 예측하는지에 대한 추가 정보가 필요함
  - 그래서 Positional Embedding 인 P를 활용함
  - 구체적으로는 아래와 같은 z_i를 활용해서 토큰을 prediction 함.
    -  \\( z_i = f(R_{s-1}, R_{e+1}, P_{i-s+1}) \\), 여기서 f는 2 layer(GeLU) fnn.


### Softmax Temperature
- <img src='https://user-images.githubusercontent.com/18374514/202433204-1d4a003b-0354-4e2b-9b0e-4f8777cc9f59.png' width='300'>
- Temperature에 따른 Softmax의 차이, T가 커질수록 smoothed 해짐
  - <img src='https://user-images.githubusercontent.com/18374514/190916755-458c4081-ea86-48c8-85d1-f046f685d864.png' width='500'>
  - By increasing the value of T, we get a smoothed probability distribution, which gives more information about other classes(이런걸 dark knowledge 라고 하는데.. 왜지?)

### Dark Knowledge를 활용한 Knowledge distillation
- First, we pre-train the teacher network with sotmax temperature to obtain dark knowledge. 
- Then, Ttransfer this dark knowledge to the student. But How?
  - <img src='https://user-images.githubusercontent.com/18374514/190917041-b2302832-2f50-4f2f-bc78-a46c3fbf531c.png' width='500'>
  - Now, we compute the cross-entropy loss between the soft target and soft prediction and train the student network through backpropagation by minimizing the cross-entropy loss(<strong>AKA distillation loss</strong>)
    -  Hard Label이 아닌 Soft Label로 Cross Entropy 계산 [Soft-Cross-Entropy 설명](https://ratsgo.github.io/insight-notes/docs/interpretable/smoothing)
  - Another, loss, **Student loss**
- ![image](https://user-images.githubusercontent.com/18374514/190919383-9adf7838-0662-4b55-b56a-014418404499.png)
  - CE with soft target (Distillation Loss)
    - ![image](https://user-images.githubusercontent.com/18374514/190919399-5951bbc0-e760-4b3a-9a1d-c7c51ed7464c.png)
  - CE with hard target(Student Loss), **argmax 있는게 다르다**
    - ![image](https://user-images.githubusercontent.com/18374514/190919404-a4b76b01-77ca-4ab1-8529-5d287fafd255.png)
    - (https://towardsdatascience.com/distilling-knowledge-in-neural-network-d8991faa2cdc)
- DistilBERT is 60% faster and its size is 40% smaller compared dto large BERT
- Apart from the distillation and student loss, we also compute **cosine embedding loss.**
- It is basically **a distance measure between the representation learned by the teacher and student BERT.** Minimizing the cosine embedding loss makes the representation of the student more accurate and similar to the teacher's embedding.

### TinyBERT
- 요약하면, 모든 레이어에 대해 pre-training, fine-tuning 시에 distllation 진행하고, Data Augmentation도 진행
- Using Knowledge distillation.
- Can we also transfer knowledge from the other layers of the teacher BERT? yes
- 단순히 Teacher의 최종 Output 뿐만 아니라 Teacher의 각 레이어의 output도 transfer 가능하다.
  - (1) Logits produced by the prediction(output) layer, (2) hidden state and attention matrix produced by the teacher BERT, (3) Output of the embedding layer 
- TinyBERT, we use a **two-stage learning framework** where we apply distillation in both the pre-training and fine-tuning stage
- TinyBERT, 4 encoder alyers, 312 hidden dimenstion, 14.5M parameters.
- TeacherBERT의Hidden Dim을 TinyBERT에 전이하기 위해서, Loss = MSE(H_teacher, H_student) 이런식으로 해야하는데, 두 모델의 Dimension이 다르므로, Linear 변환을 위한 W를 H_teacher에 곱해준다. Embedding도 마찬가지. W는 당연히 학습됨.
- Prediction Layer distilation by minimizing the cross-entropy loss between the soft arget and soft prediction.
  - L_pred = -softmax(Z_t) * log_softmax(Z_s) 로 학습됨. Z는 각 모델의 output logit
- Final Loss, summation of belows
  - <img src='https://user-images.githubusercontent.com/18374514/202441554-189ad2c0-3deb-4133-ba40-4bc3a0b7189b.png' width='300'>
- In TinyBERT, two-stage learning
  - 1) General distilation : Pretraining step. 
  - 2) Task-specific distillation : Fine-tuned BERT로 distillation.
#### Data Augmentation in TinyBERT
- Distillation시에 데이터가 많이 필요하기 때문에 DA가 필수인듯?
- 1) Single-piece word 만 Mask 한 뒤, K개의 Prediction candidates를 뽑음(BERT 사용)
- 2) Sinilge-Piece가 아니면 마스크 하지 않음. 대신 glove embedding으로 가장 유사한 단어를 찾아서 candidate로 만듦
- 3) 그 다음, 일정 Threshold(예를 들어 0.4) 기준으로 단어를 candidates으로 교체하거나 그대로 둠.

#### BERTSUM 에서 놓치면 안되는것.
- Extractive Summarization의 경우, 모든 Sentence 앞에 \[CLS\] 토큰을 입력
- Segment EMbedding의 경우에는 Interval 하게 $$E_A$$, $$E_B$$, $$E_A$$ 이런식으로 번갈아서 주면 된다
- Position Embedding 은 동일

##### BERTSUM with an inter-sentence transformer
- BERT의 Sentence Output에 2-layer Transformer를 얹음. Positional Embedding만 적용
- LSTM도 얹는게 가능. 
- LR= 2 $$e^{-3}$$ * min( $$step^{-0.5}$$, $$step * warmup^{-1.5}$$), warmup=10000

#### Abstracitve summarization using BERT
- Decoder는 Random init 이기 때문에 Encoder랑 다른 LR, Optimizer를 가져감
- Encoder보다 Decoder가 훨씬 LR이 큼. 동일한 LR을 사용하면 Encoder는 overfit, Decoder는 Underfit 될 확률이 있음.

#### ROUGE Metric
- Recall-Oriented Understudy(대역) for Gisting(요점 추리기) Evaluation
- Recall = $$Total number of overlapping n-grams\over Total number of n-grams in the reference summary$$
- ROUGE-1 : a unigram recall
  - Candidate Summary : Machine learning is seen as a subset of artificial intelligence.
    - unigrams : manchine, learning, is, seen, as, a, subset, of, artificial, intelligence
  - Reference summary : Machine learning is a subset of artificial intelligence.
    - unigrams : machine, learning, is, a, subset, of, artificial, intelligence
  - Recall = 8/8 = 1
- ROUGE-2 : a bigram recall
  - candidate summary bigrams : (mchine learning), (learning is), (is seen), (seen as), (as a), (a subset), (subset of), (of artifical), (artificial intelligence)
  - reference summary bigrams : (machine learning), (learning is), (is a), (a subset), (subset of), (of artifical), (artificial intelligence)
  - recall = 6/7 = 0.85
- **ROUGE-L** : the longest common subsequence(LCS), ROUGE-L is calculated using the **F-measure.**
  - $$R_{lcs}$$ = LCS(candidate, reference)/Total number of words in the reference summary.
  - $$P_{lcs}$$ = LCS(candidate, reference)/Total number of words in the candidate summary.
  - $$F_{lcs}$$ = $$(1+b^2)R_{lcs}P_{lcs} \over R_{lcs} + b^2P_{lcs}$$, b is used for controlling the importance of precision and recall.

### Applying BERT to Other Languages
- NLI 데이터셋 종류: SNLI(Stanford Natural Language Inference), MLNI(Multi-Genre natural Language Inference), XNLI(Cross-lingual NLI)
  - XLNI는 MNLI파생이기 때문에 겹치는게 있다(영어)
#### How multilingual is multilingual BERT?
- **Effect of Vocabulary Overlap**
  - 만약 Vocab이 겹치는게 많다면, Vocab 이 많이 겹치는 언어로의 Zero-shot Transfer 성능이 좋아야 한다.
  - <img src='https://user-images.githubusercontent.com/18374514/202898656-6c30dea3-14b0-43f3-aae0-96055a4cc8aa.png', width='500'>
  - 위 결과를 보면 그렇지는 않았다. 그러므로, 단순히 memorizaing vocabulary 하는 건 아니다.
- **Generalizatoin across scripts(글씨체)**
  - Urdu와 Hindi는 각각  서로 다른 scripts를 갖는다. 그러나 Urdu언어로 된 POS Task에 fine-tuned 된 모델은 Hindi에도 잘 동학한다.
  - This indicates that M-BERT has mapped the Urdu annotations to the Hidi words.
  - Scripts 가 달라도 동작하는건 신기. 근데 English-Japanese 사이에서는 동작하지 않았음. 그러니까 Typological similarity가 전혀 없는 영어-일본어 등은 동작하지 않는 것으로 봐서 Sripts 가 달라도 동작하는 것은 Typological similarity 때문으로 파악됨
- **Generalization across typological features.
  - Bulgarian과 영어는 동일한 Word order를 갖는다(주어 동사 목적어 등). 따라서 완전히 다른 일본어보다 Bulgarian에서 POS Task에서 영어로 학습한 모델의 zero-shot 성능이 더 좋게 나왔다.
- **Effect of language similarity**
  - <img src='https://user-images.githubusercontent.com/18374514/202899049-9d7a6ccb-f300-4966-bb93-d41a166c788e.png' width='500'>
  - WALS : world atlas of language structures, 언어간 특성 파악 database
  - 즉, 언어학적으로 유사한 언어들끼리의 Transfer 성능이 잘 나옴.
- **Effect of code switching and transliteration**
  - Code Switching? 서로 다른 언어를 Mixing 한것. "Hi, 나는 현제야"
  - Transliteration? 발음 그대로 언어를 옮긴 것. "하이, 아이엠 현제야"
  - 이 부분 설명이 부실해서, 명확히 이해를 못했으나, 어쨌든 Code-Swiching은 가능하나 Transliteration은 Transfer learning이 잘 안된다.
 
#### The cross-lingual language model(cross-lingual objective)
- The XLM(cross-lingual language model) is pre-trained using the monolingual and parallel datasets.
- Parallel datasets : MultiUN, OPUS, IIT Bombay corpus.
- XLM uses byte pair encoding

##### Pretraining strategies
- Causal Language modeling(CLM) : $$P(w_t|w_1,w_2,...,w_{t-1};\theta)$$
- MLM : BERT의 MLM과 다른 점은 Sentence Pair로 입력하지 않는 것, 그리고 token length 256. 자주 등장하는 token과 rare token의 밸런스를 맞추기 위해 squre root of inverse frequency를 이용
- TLM: Source-Target 언어를 함께 넣고 MLM
- XLM : MLM+TLM
- XLM-R : MLM Only

#### Sentence-BERT with triplet loss
- <img src='https://user-images.githubusercontent.com/18374514/202901601-e41f4cf1-1c44-43c4-abbb-7b7392adea64.png' width='500'>
- 입실론은 margin을 의미, 이것은 positive sentence가 최소한 입실론 만큼은 negative보다 가까워지게 만들어줌. **논문에서는 1로 셋팅**
- loss function이니까 max(.,0) 해서 음수로는 안되게하고.. metric은 euclidean으로 함

#### Learning language and video representations with VideoBERT
- <img src='https://user-images.githubusercontent.com/18374514/202905359-e9ac7348-37c8-4b63-bf84-afdeec483f06.png' width='500'>
  - 위와 같은 비디오에서 representation을 배우는 법
  - ASR로 Language Token은 구함
  - 비디오에서 1.5초동안 20fps의 이미지를 샘플링하여 Image token을 구함
  - 이 둘을 Concat, random mask, add special token, then
  - <img src='https://user-images.githubusercontent.com/18374514/202905467-fde19cdd-57a6-48ac-98b8-c5680594f8fe.png' width='500'>
  - <img src='https://user-images.githubusercontent.com/18374514/202905592-78e9ea77-0973-4aab-8a06-20c21bd3937e.png' width='500'>
  - 그리고 아래 그림처럼, **이미지와 텍스트가 Align 되는지도 학습함**
  - <img src='https://user-images.githubusercontent.com/18374514/202905743-d26dd069-5d5f-4df0-8547-40529975660c.png' width='500'>
- The final pre-training objectives
  - 1) Text-only : just MLM
  - 2) Video-only : visual token으로 MLM
  - 3) text-video : image token과 language token을 모두 MLM, Align Task도 수행( 같이 한다는건가?)
  - The final -pretraining objective of VideoBERT is the weighted combination of all of the three methods.
  - 2days using 4 TPUs for 0.5M iterations.
- Dataset
  - 유투브에서 요리 instruction과 관련된 동영상 모음. 312,000 개의 23,186 hours or 966 days
  - 텍스트는 Youtube API 활용
  - 자막이 안달리는 영상은 Video-Only objective에 활용하고, 나머지는 다 사용.
- Application
  - <img src='https://user-images.githubusercontent.com/18374514/202906114-ad70c991-b219-431d-88ab-7a0b60ecb54c.png' width='500'>

#### Understanding BART
- Text Infiling과 SpanBERT와 다른점
  - 일정 Span Tokens을 MASK로 치환하는건 동일하나, 예를 들어 3개의 토큰을 치환하더라도 BART에서는 하나의 [MASK]로 변경 하고, SpanBERT는 3개로 변경
  - 근데 코드로 구현하면, Text Infiling에서 몇개의 토큰으로 변경할지는 어떻게 구현하지?

# Edit
- https://github.com/oglee815/oglee815.github.io/edit/master/_posts/1900-01-01-book-getting-started-with-bert.md
