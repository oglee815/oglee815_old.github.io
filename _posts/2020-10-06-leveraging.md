---
layout: post
date: 2020-10-06 19:57
title: Leveraging Pre-trained Checkpoints for Sequence Generation Tasks
author: Sascha Rothe
description: Leveraging Pre-trained Checkpoints for Sequence Generation Tasks
comments: true
math: true
category: 
- NLP
tags:
- Pretrained Model
- NLG
- Google
---

NLG Task 에서 PLM을 이용하는 방법에 관한 논문

 <!--more-->

## Abstract
> Unsupervised pre-training of large neural models has recently revolutionized Natural Language Processing. By warm-starting from the publicly released checkpoints, NLP practitioners have pushed the state-ofthe-art on multiple benchmarks while saving signiﬁcant amounts of compute time. So far the focus has been mainly on the Natural Language Understanding tasks. In this paper, we demonstrate the efﬁcacy of pretrained checkpoints for Sequence Generation. We developed a Transformer-based sequence-to-sequence model that is compatible with publicly available pre-trained BERT, GPT-2 and RoBERTa checkpoints and conducted an extensive empirical study on the utility of initializing our model, both encoder and decoder, with these checkpoints. Our models result in new state-of-the-art results on Machine Translation, Text Summarization, Sentence Splitting, and Sentence Fusion.

## Introduction

- Unsupervised, self-supervised pretraining methods들은 NLU분야에서 또 다른 수준의 Benchmark 퍼포먼스를 보여주었다.
- 이러한 부분에서 가장 매력적인 부분은 거대한 모델을 대량의 데이터로 학습한 pre-trained 모델의 checkpoint와 inference code가 freely available 한 것이다.
- 덕분에 많은 연구자들이 pre-training 에 필요한 대량의 리소스를 줄이면서도 fine-tunning 성능은 대폭 향상 시킬 수 있었다.
- 하지만 대부분 NLU Task에만 사용되었고 Seq-to-seq 과 같은 NLG Task에 warm-start 용도로 사용되지는 않았었다.
- 우리는 이 논문에서 <span class='my_highlight'>publicly available pre-trained BERT, GPT-2 and RoBERTa checkpoints를 이용하여 seq-to-seq을 위한 모델을 만드는 방법</span>을 제안한다.
- We aim to provide an empirical answer to the following research question: <span class='my_highlight'>what is the best way to leverage publicly available pre-trained checkpoints for warm-starting sequence generation models? </span>
- 예를 들어, BERT Encoder로 input을 이해하는데 사용하고 GPT Decoder를 Text Generation에 사용 할 수 있을것이다.
- BERT, GPT, RoBERTa의 다양한 조합에 대한 실험을 진행하였다.
- 평가에 진행한 Task는 sentence-level fusion, splitting, MT, abs sum(CNN/DM, Gigaword, BBC extreme)
- 우리 제안 모델이 Randomly initialized 모델보다 높은 성능을 보였다.
- a pre-trained encoder is an essential component for sequence generation tasks and often these tasks benefit from sharing the weights between the encoder and the decoder

## Models And Pre-trained Checkpoints
- BERT는 원본의 구조대로 구현(GELU)
- Decoder는 BERT와 거의 동일하나 Masked Attention과 encoder-decoder attention으로 구현
- Note, that if the model was randomly initialized, we found no difference between a BERT compatible decoder and a GPT-2 compatible decoder. 
  > Pretrained 되지 않은 조건하에, 디코더를 버트로 하나 GPT2로 하나 결과가 크게 상관 없었다는 얘기?
- 학습된 체크포인트를 쓰므로 기본적으로 12layer 버전의 구조를 따라가나, 베스트는 24layer를 가진 체크포인트 였다.(24 layers, a hidden size of 1024, ﬁlter size of 4096, and 16 attention heads)
- For fine-tuning
  -  Adam(0.05), a linear learning rate warmup with 40k steps, normalization by the square root of the hidden size, and a square root decay.
  - The batch size and the number of training steps will be reported for each task individually. 
- **BERT Checkpoints**
  - WordPiece Tokenizer, BERT-Base Cased, BERT-Base Uncased, BERT-Base Multilingual Cased
- **GPT-2 Checkpoints**
  - SentencePieces Tokenizer,
  - We count 125M parameters in the checkpoint.
  - The vocabulary size is ∼50k. While GPT-2 has positional embeddings for up to 1024 position, **we only use the ﬁrst 512 to make the results comparable with BERT**
- **RoBERTa Checkpoints**
  - The vocabulary treatment in RoBERTa is compatible with the SentencePiece tokenization in GPT-2

## Investigated Model Variants
- **RND2RND** : all weights initialized randomly.
- **BERT2RND** : A BERT-initialized encoder paired with a randomly initialized decoder. Encoder and decoder share the embedding matrix initialized from a checkpoint. 
- **RND2BERT** : A randomly initialized encoder paired with a BERT-initialized decoder. We mask the bidirectional self-attention mechanism of BERT to look only at the left context. 
- **BERT2BERT** : A BERT-initialized encoder paired with a BERT-initialized decoder. The only variable that is initialized randomly is the encoder-decoder attention.
- **BERTSHARE** : Like BERT2BERT, but the parameters between encoder and decoder are shared. Param Share 덕분에 221M -> 136M 으로 사이즈 줄어듬. Layer-Wise Attention 도 실험했으나, 큰 차이는 없었음.
  > 
- **ROBERTASHARE** : Same with BERTSHARE, but with ROBERTA checkpoints
- **GPT** : A decoder-only architecture. The decoder is warm-started with a public GPT-2 checkpoint. 
- **RND2GPT** : A randomly initialized encoder paired with a GPT-2-compatible decoder
- **BERT2GPT** : A BERT-compatible encoder paired with a GPT-2-compatible decoder. 
We warm-start both sides with the two separate, BERT and GPT-2, public checkpoints. We use the BERT vocabulary for the input and the GPT-2 vocabulary for the output. 
- **ROBERTA2GPT** Same as BERT2GPT, but we use a public RoBERTa checkpoint to warm-start the encoder. <span class='my_highlight'>RoBERTa was trained using the GPT2 vocabulary so we can use it for input and output.</span> Note that while the <span class='my_highlight'>vocabulary is shared</span>, this model still has <span class='my_highlight'>two embeddings matrices</span>, one for the input and one for the output. 
  > Pretrained 될 당시의 Vocab하고 다른데 어떻게 RoBERTa를 GPT2 Vocab으로 Finetune 한걸까..?
- BERT나 RoBERTa가 bidirectional representation을 배우도록 학습되지만, 디코더로 쓰일 때는 autoregressive fashion으로 Generate 함.
- BERT나 RoBERTa는 24layers에 대한 결과도 보여주나, GPT2는 24layer 버전에서 결과가 좋지 않아서 제외함.

## Experiments and Results

### Sentence Fusion
- Sentence Fusion is the problem of **combining multiple sentences into a single coherent sentence.**
- "balanced Wikipedia" portion of the DiscoFuse dataset (4.5M for training, 50K for test)
<span class='centered'>![discoFuse](/assets/img/DiscoFuse.png)</span>
- 300k steps, a global batch size 256. The input and output are padded to a length of 128, which covers 100% of the training, evaluation and test data. 
- We report <ins>[SARI](https://cocoxu.github.io/publications/tacl2016-smt-simplification.pdf)</ins> and the exact match accuracy.
<span class='centered_big'>![discoFuseResult](/assets/img/DiscoFuse_result.png)</span>
- ROBERTA2GPT, ROBERTASHARE 가 가장 좋았음.(SOTA 이김)

### Split and Rephrase
- The **reverse** task of sentence fusion is the split-and-rephrase task, which requires **rewriting a long sentence into two or more coherent short sentences.**
- 1M examples of sentence splits extracted from the Wikipedia edit history
- 300k steps with a global batch size of 256.
- The input and output are padded to a length of 128, which covers 100% of the training, evaluation and test data.
- report corpus-level BLEU, the exact match accuracy, and SARI score
<span class='centered_big'>![WikiSplitResult](/assets/img/WikiSplit_result.png)</span>
- **Share 모델이 전반적으로 좋았음**

### Machine Translation
- most common benchmark in machine translation – **WMT 2014 English ↔ German task** – using newstest2014 and newstest2016 eval sets
- same hyperparameter settings as in the previous experiments
- i_l, o_l = 128, b=256, 30 epochs, 
- Decoding beam size of 4 and the default value for the sentence length penalty is set to α = 0.6
- uncased BLEU-4 score
<span class='centered_big'>![MT_leveraging.png](/assets/img/MT_leveraging.png)</span>
- BERT checkpoint receive a signiﬁcant boost: BERT2RND >>> RND2RND
  > 논문 쓸 당시에 XLM-R을 못찾은듯.
- Contrary to the WikiSplit and DiscoFuse task, **sharing the encoder and decoder variables did not give an additional boost.** This is most likely because a) **model capacity** is an important factor in MT and b) encoder and decoder have to deal with **different grammar and vocabulary**
- GPT는 주로 영어 코퍼스에서 훈련이 되었기 때문에 MT에서는 성능이 구리게 나옴. 그래서 회색으로 표시됨.
- **Customized BERT checkpoint** : 기존에 제공되던 104 languages M-BERT는 성능이 WMT14에서 너무 구리게 나옴. 그래서 다음과 같이 커스터마이징
  - 32k wordpiece vocabulary extracted from the WMT14 En ↔ De training set 
  - pre-train a BERT model on the English and German subset of the Wikipedia dump
  - 훨씬 좋은 결과 나옴.

### Abstractive Summarization
- 3 Tasks : Gigaword, CNN and DailyMail, and BBC extreme
  - The Gigaword : 3.8M sentence-summary training pairs, lowercased version, Doc 128 tokens, sum 32 tokens
  - CNN/DailyMail : 287k document-summary pairs, 거의 extract에 가까움. original cased versions, Doc 512 tokens, sum 128 tokens
  - BBC dataset : 204k single sentence summaries, extream abstractive, Doc 512 tokens, sum 64 tokens
