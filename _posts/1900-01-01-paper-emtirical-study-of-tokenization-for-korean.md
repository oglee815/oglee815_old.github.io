---
layout: post
date: 2020-11-02 14:00
created_date: 2020-11-02 14:00
title: "[Paper] 2010 An Empirical Study of Tokenization Strategies for Various Korean NLP Tasks"
author: Kyubyong Park et.al @Kakao Brain
description: An Empirical Study of Tokenization Strategies for Various Korean NLP Tasks
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Tokenization
- 형태소
- Korean
- Paper
---

Tokenization 방법에 따른 한국어 NLU Task 성능 비교

 <!--more-->

## Abstract
> Typically, tokenization is the very first step in most text processing works. 
As a token serves as an atomic unit that embeds the contextual information of text, how to define a token plays a decisive role in the performance of a model.
Even though Byte Pair Encoding (BPE) has been considered the de facto standard tokenization method due to its simplicity and universality, it still remains unclear whether BPE works best across all languages and tasks. 
In this paper, we test several tokenization strategies in order to answer our primary research question, that is, “What is the best tokenization strategy for Korean NLP tasks?”
Experimental results demonstrate that a hybrid approach of morphological segmentation followed by BPE works best in Korean to/from English machine translation and natural language understanding tasks such as KorNLI, KorSTS, NSMC, and PAWS-X. As an exception, for KorQuAD, the Korean extension of SQuAD, BPE segmentation turns out to be the most effective.
Our code and pre-trained models are publicly available at https://github.com/kakaobrain/kortok.
> Tokenization에 따라서 Embedding 결과가 달라지므로 굉장히 중요하다.
> 한국어에 있어서는 BPE + 형태소 분석기를 활용한 하이브리드 방식이 가장 좋았다.

## Introduction
- Tokenization은 워낙 중요하기 때문에 많이 연구되어왔는데, 이중 BPE가 가장 인기가 많음.
- 그 이유는 Language independent 이기 때문.
- 하지만 모든 언어에 최적일리는 없음.
- 따라서 우리는 한국어에 대한 토크나이제이션 메소드에 대해서 연구한다.

## Background
- MeCab-ko: A Korean Morphological Analyzer
  - 오픈소스 형태소 분석기. Conditional Random Fields에 기반함.
  - 세종 코퍼스에 대해서 훈련됨.
- BPE(Byte Pair Encoding)
  - 간단한 데이터 압축 기술, 가장 많이 등장하는 pair of bytes를 single, unsued byte로 대체함.

## Related Work
- Several papers claimed that a hybrid of linguistically informed segmentation and a data-driven method like BPE or unigram language modeling performs the best for non-English languages.
- Moon and Okazaki (2020) proposed a novel encoding method for Korean and showed its efficiency in vocabulary compression with a few Korean NLU datasets.
> 어떤 방법이지?

## Tokenization Strategies
- Consonant(자음) and Vowel(모음) (CV)
- Syllable(음절)
- Morpheme(형태소)
- Subword
  - SentencePiece 토크나이저를 사용
- Morpheme-aware Subword
  - MeCab-ko and BPE in sequence to make morpheme-aware subwords
  - 먼저 형태소 단위로 나눈 다음 BPE를 수행. 따라서 "나랑 쇼핑하자"의 문장의 경우, '핑하' 와 같이 단순히 많이 등장한다고 해서 tokenizing되는 경우는 없음.
- Word : White Space 단위

## Experiments
- Dataset
  - KO-EN parallel corpus from AI HUB(800K)
- BPE Modeling
  - 학습 데이터로 무엇을 사용할 것인가? 1) AI HUB news 2) Wiki
    - Wiki는 사이즈는 크지만 Test Data Set과 차이가 있음
    - AI hub news 데이터는 사이즈는 작지만(130MB) Test Set과 매우 흡사
    - (A) AI HUB 데이터로 훈련함(32K BPE model with SentencePiece)
    - (B) EN Wiki, Ko Wiki 데이터로 훈련함 32K BPE model
    - A,B 모델을 en-ko, ko-en 데이터로 번역 훈련함.

## Training
- Transformer, 6 blocks of 512-2048 units with 8 attention heads. 
- Each model is trained using a Tesla V100 GPU with batch size 128, dropout rate 0.3, label smoothing 0.1, and the Adam optimizer.

## Result sumup
- To sum up, morpheme-aware subword tokenization that makes the best use of linguistic knowledge and statistical information is the best for Korean machine translation.