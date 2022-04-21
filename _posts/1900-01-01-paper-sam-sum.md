---
layout: post
date: 2021-02-02 14:00
created_date: 2021-02-02 14:00
title: "[Paper] 1911 SAMSum Corpus- A Human-annotated Dialogue Dataset for Abstractive Summarization"
author: Bogdan Gliwa, Iwona Mochol, Maciej Biesek, Aleksander Wawer @ Samsung R&D Institute Poland
description: SAMSum Corpus- A Human-annotated Dialogue Dataset for Abstractive Summarization
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Abstractive Summarization
- Dialogue Summarization
- Dataset
- Paper
---

The SAMSum dataset contains about 16k messenger-like conversations with summaries. 

 <!--more-->

## Abstract
> This paper introduces the SAMSum Corpus, a new dataset with abstractive dialogue summaries. We investigate the challenges it poses for automated summarization by testing several models and comparing their results with those obtained on a corpus of news articles. We show that model-generated summaries of dialogues achieve higher ROUGE scores than the model-generated summaries of news – in contrast with human evaluators’ judgement. This suggests that a challenging task of abstractive dialogue summarization requires dedicated models and non-standard quality measures. To our knowledge, our study is the first attempt to introduce a high-quality chatdialogues corpus, manually annotated with abstractive summarizations, which can be used by the research community for further studies.

##  Introduction and related work
- In the abstractive approach important pieces of information are presented using words and phrases not necessarily appearing in the source text. This requires natural language generation techniques with **high level of semantic understanding**
- One of the reasons is the availability of large, high-quality news datasets with annotated summaries, e.g., **CNN/Daily Mail**
- **Such a comprehensive dataset for dialogues is lacking.**
- AMI 데이터 셋이 Speech 쪽에는 있다. 다만 그 양이 매우 적다(141 dialogues)
- 그래서 우리가 대화 데이터셋을 만들겠다!!!!!야호!! 16K

##  SAMSum Corpus
- we considered two approaches to build it: 
  - (1) using existing datasets of documents, which have a form similar to chat conversations, 
    - But, they were too technical, too long , lacked context  or they were more of a spoken type --> give up
  - (2) creating such a dataset by linguists.
- **Process of building the dataset.**
  - 더 대화체적이고, 영어를 잘하는 한명의 언어학자에 의해 만들어짐.
  - slang, emoticon, typo 등이 포함
- **Validation**
  - We asked two linguists to doubly annotate 50 conversations in order to verify whether the dialogues could appear in a messenger app and could be summarized (i.e. a dialogue is not too general or unintelligible) or not 
  - 94%는 괜찮다고 판단
- **Cleaning**
  -  a colon should separate an author of utterance from its content, each utterance is expected to be in a separate line. 
  -  이름 틀린거는 고쳤음
- **Description**
  - total **16369**, conversations distributed uniformly into 4 groups based on the number of utterances in conversations: **3-6, 7-12, 13-18 and 19-30.**
  - 대부분 화자는 2명
  - <span class='centered_small'>![dataset](/assets/img/samsum_dataset.png)</span>
  - <span class='centered_small'>![dataset](/assets/img/samsum_example.png)<span>

## Dialogues baselines
- The baseline commonly used in the news summarization task is **Lead-3**, which takes **three** leading sentences of the document as the summary
- Inspired by the Lead-n model, we propose a **few different simple models**: 
  - **MIDDLE-n**, which takes n utterances from the middle of the dialogue,
  - **LONGEST-n**, treating only n longest utterances in order of length as a summary,
  - **LONGER-THAN-n**, taking only utterances longer than n characters in order of length (if there is no such long utterance in the dialogue, takes the longest one),
  - **MOST-ACTIVE-PERSON**, which treats all utterances of the most active person in the dialogue as a summary.
  - <span class='centered_small'>![dataset](/assets/img/samsum_baseline.png)<span>
  - Nevertheless, the best dialogue baseline turns out to be the **LONGEST-3 model.**

## Experimental setup
### Data preparation
- In order to build a dialogue summarization model, we adopt the following strategies: 
  - (1) each candidate architecture is trained and evaluated on the dialogue dataset; 
  - (2) each architecture is trained on the train set of **CNN/DailyMail joined together** with the train set of the dialogue data, and **evaluated on the dialogue test set.**
- utterances are separated with a special token called the separator 
  - artificially added token e.g. \<EOU\> for models using **word embeddings**, "\|" for models using **subword embeddings.**
  - news and dialogues are truncated to **400** tokens, and summaries – to **100** tokens. The maximum length of generated summaries was not limited.

### Models
Beam size for beam search decoding to **5**
- **Pointer generator network** : In the case of Pointer Generator, we use a default configuration, changing only the **minimum length of the generated summary from 35 (used in news) to 15 (used in dialogues).**
- **Transformer** : The model is trained using **OpenNMT** library. We use the same parameters for training both on news and on dialogues, changing only the **minimum length of the generated summary – **35 for news and 15 for dialogues.**
- **Fast Abs RL** : It is trained using its default parameters. For dialogues, we change the convolutional word level sentence encoder (used in extractor part) to only use kernel with size equal 3 instead of 3-5 range. **It is caused by the fact that some of utterances are very short and the default setting is unable to handle that.**
- **Fast Abs RL Enhanced** : each utterance, at the end, after artificial separator, we **add names of all other interlocutors**
  - The reason for that is that **Fast Abs RL requires text to be split into sentences** (as it selects sentences and then paraphrase each of them).
  - For dialogues, we divide text into utterances (which is a natural unit in conversations), so sometimes, a single utterance may contain more than one sentence. Taking into account how this model works, it may happen that it selects an utterance of a single person (each utterance starts with the name of the author of the utterance) and has no information about other interlocutors (if names of other interlocutors do not appear in selected utterances), so it may have no chance to use the right people’s names in generated summaries.
  > Fast Abs Sum이 동작하는 방식이, Sentence를 선택하고 Paraphrasing을 하는 방식인데, Utterance는 한 Unit에 여러 문장이 있다. 따라서 다른 문장에 등장하는 화자의 이름을 이용해서 Sum을 해야하면 Model은 이름을 알지 못하므로 정확도가 떨어질 수 밖에 없기 때문에, 각 Utt에 모든 사람의 이름을 추가한다.
- **LightConv and DynamicConv** : 
- The implementation is available in fairseq. We train lightweight convolution models in two manners: 
  - (1) learning token representations from scratch; in this case we apply BPE tokenization with the vocabulary of 30K types, using fastBPE implementation 
  - (2) initializing token embeddings with pre-trained language model representations; as a language model we choose **GPT-2 small**.
  
### Evaluation metrics
- ROUGE-1, ROUGE-2 and ROUGE-L using the py-rouge package

## Results
- News Dataset 에서는 **Lead-3를 이긴 Model은 Fast Abs RL 이 유일함**
  - <span class='centered_small'>![dataset](/assets/img/samsum_news_eval.png)<span>
- In general, the **Transformer-based architectures benefit from training on the joint dataset: news+dialogues**, even though the news and the dialogue documents have very different structures.
- **The inclusion of a separation token between dialogue utterances is advantageous for most models** – presumably because it improves the discourse structure. The improvement is most visible when training is performed on the joint dataset.
- For Fast Abas RL Model, **enhancing utterances** with information about the other interlocutors helps achieve higher ROUGE values.
- The **largest improvement** of the model performance is observed for LightConv and Dynamic-Conv models when they are complemented with pretrained embeddings from the language model **GPT-2**, trained on enormous corpora.
- It is also worth noting that some models (Pointer Generator, Fast Abs RL), trained only on the dialogues corpus (which has 16k dialogues), reach similar level (or better) in terms of ROUGE metrics than models trained on the CNN/DM news dataset (which has more than 300k articles). Adding pretrained embeddings and training on the joined dataset helps in achieving significantly higher values of ROUGE for dialogues than the best models achieve on the CNN/DM news dataset.
> Dialogue 만 학습한 친구들이 CNN/DM도 함께 학습한 애들하고 비슷하거나 더 좋은 경우도 있다. GPT Emb 더해서 News + Dialogue를 함께 학습하는게 Best
  - <span class='centered_small'>![dataset](/assets/img/samsum_final_eval.png)<span>

## Linguistic verification of summaries
- n-gram 단위로 겹치는 단어의 점수를 파악하는 Rouge는 Metric으로 좋지 않으므로, 언어학자들에게 150 news and 100 dialogues 에 대해서 평가하도록 했음 (점수 : −1, 0, 1)
- we calculated the **linear weighted Cohen’s kappa coefficient** (McHugh, 2012) between annotators’ scores.
  - For news examples, we obtained agreement on the level of **0.371 and for dialogues – 0.506**
    > Dialogue 에서 두 전문가의 의견이 더 비슷한데, 이는 대화가 뉴스보다 짧고 쉽기 때문이라고 예측
- Model이 만든 대화 요약이 뉴스요약보다 Rouge 점수는 높지만 human Eval은 훨씬 후지다. 
  > Rouge 점수는 대화 요약 결과보다 뉴스 요약 결과에 더 잘 매칭한다.
- <span class='centered_small'>![dataset](/assets/img/samsum_human_eval.png)<span>
- - <span class='centered_small'>![dataset](/assets/img/samsum_human_eval_corr.png)<span>

## Difficulties in dialogue summarization
- 정보의 흐름이 매우 명확한 뉴스와 달리, 대화는 discussion, question, answer, greetings 등 중요한 정보가 다른 화자들의 대화에 흩뿌려져 있다.
- 그리고 Article은 제 3자의 시점에서 쓰여졌고, chat은 모두가 그들 스스로에 대해서 말하고, 대명사도 다양하다. 
- 또한 메신저에서 사람들이 말할 때, words 를 줄이거나 slang을 사용한다. 오타도 있고.
- Model의 문제점
  - 이름과 행동의 관계 파악, 이름을 반복함. 누가 무슨 행동을 했는지 주어와 목적어 등을 헷갈림
  - Fast Abs 와 같은 모델은 Utterence를 먼저 선택하고 Paraphrasing 을 하기 때문에 Context를 잃어버릴 때가 있다.
  
## Discussion
- This paper is a step towards abstractive summarization of dialogues by 
  - (1) introducing a new dataset, created for this task, 
  - (2) comparison with news summarization by the means of automated (ROUGE) and human evaluation.
- In terms of human evaluation, the results of **dialogues summarization are worse than the results of news summarization**
- This is connected with the fact that the dialogue structure is more complex – 
**information is spread in multiple utterances, discussions, questions, more typos and slang words appear there, posing new challenges for summarization.**
- We demonstrate in experiments that the models benefit from **the introduction of separators**, which mark utterances for each person.
- We show that the most popular summarization metric **ROUGE does not reflect the quality of a summary.**
  - Rouge 기준으로는 Dialogue Sum이 CNN/DM News Sum보다 높으나, Human Eval에서는 훨씬 안좋았음.
- We conclude that when measuring the quality of modelgenerated summaries, the ROUGE metrics are more indicative for news than for dialogues, and a new metric should be designed to measure the quality of abstractive dialogue summaries.
    > 반드시 다른 Metric이 필요하다.

## Conclusions
- In our paper we have studied the challenges of **abstractive dialogue summarization**
- The next step could be creating an even more challenging dataset with **longer dialogues** that not only cover one topic, but span over numerous different ones.