---
layout: post
date: 2022-02-23 14:00
created_date: 2022-02-23 14:00
title: "[Paper] 2111 FLEX, Unifying Evaluation for Few-Shot NLP"
author: Jonathan Bragg, Arman Cohan, et al. @ Allen AI
description: Unifying Evaluation for Few-Shot NLP
comments: true
math: true
category: 
- Paper
tags:
- NLP
- Few-Shot
- leaderboard
---

Few-shot을 위한 Dataset 공개. Prompt engineering의 복잡성을 QA 문제로 전환함으로써 해결(4가지 타입의 QA templates로 prompt template 사용)
<!--more-->

## Abstract
> Few-shot NLP research is highly active, yet conducted in disjoint research threads with evaluation suites that lack challenging-yet-realistic testing setups and fail to employ careful experimental design. Consequently, the community does not know which techniques perform best or even if they outperform simple baselines. In response, **we formulate the FLEX Principles, a set of requirements and best practices for unified, rigorous, valid, and cost-sensitive few-shot NLP evaluation.** These principles include Sample Size Design, a novel approach to benchmark design that optimizes statistical accuracy and precision while keeping evaluation costs manageable. Following the principles, we release the FLEX benchmark,2 which includes **four few-shot transfer settings, zero-shot evaluation, and a public leaderboard that covers diverse NLP tasks.** In addition, we present **UniFew**, **a prompt-based model for few-shot learning** that unifies pretraining and finetuning prompt formats, eschewing complex machinery of recent prompt-based approaches in adapting downstream task formats to language model pretraining objectives. We demonstrate that despite simplicity, UniFew achieves results competitive with both popular meta-learning and prompt-based approaches.

## Intro
- Problem
  - despite the shared goal of advancing few-shot NLP techniques, the community does not know **which techniques work best or even if they perform better than simple baselines.**
  - Evaluation suites across these research threads are disjoint, **lack challenging-yet-realistic testing setups** (e.g., class imbalance, variable training set sizes, etc.), and do not employ careful experimental design to ensure accurate and precise evaluation estimates and minimal computational burden.
  - Need for systematic benchmark design
  - Need for a robust few-shot model : Can models eschew(피하다) **complex prompt engineering** by unifying pretraining and downstream task formats?
- Contributions
  - FLEX Principles: a set of requirements and best practices for few-shot NLP evaluat
  - FLEX benchmark: an implementation of the FLEX Principles. It tests across four few-shot transfer settings
  - UniFew: a prompt-based model for few-shot learning in NLP
    - UniFew, is to close the gap between pre-training and fine-tuning formats by posing tasks **as multiple-choice QA** and using an underlying model that is pre-trained on a similar natural QA task format.

## Background
- 3 phase of evaluation procedure in modern approaches
  - 1) a general-purpose pretrained model is obtained.
  - 2) **meta-training phase**, techniques aim to adapt the model to be well-suited for few-shot generalization
  - 3) **meta-testing phase**, evaluates the adapted model in new few-shot prediction settings
- Few-shot evaluation in NLP
  - **class transfer**
  - **domain transfer** keeps the same classes between meta-training and meta-testing (e.g. **SciTail** : The SciTail dataset is an entailment dataset created from multiple-choice science exams and web sentences.)
  - **task transfer**, the problem of generalizing from a set of tasks at meta-train time to **unseen tasks at meta-test time**
  - **pretraining transfer** : Prior work has shown that pretrained language models are capable of surprising performance on many few-shot tasks, even **without fine-tuning(GPT3)**

## FLEX Principles for Few-Shot NLP Evaluation
- Diversity of transfer types: The benchmark should measure all four transfer settings(위에서 언급한 4가지)
- Variable number of shots and classes: variety of training set sizes and numbers of classes
- Unbalanced training sets:  The benchmark should also include unbalanced training sets with different training shots per class(class imbalance)
- Textual labels
- Zero-shot evaluation 
- No extra meta-testing data
- Principled sample size design : We believe the benchmark’s test sample size should be optimized to enable proper performance evaluation for such techniques, while ensuring the computational burden is inclusive(포함된) toward researchers without large compute resources.
- Proper reporting of CIs, SDs, and individual results

## UniFew: A Few-shot learning Model by Unifying pre-training and downstream task formats
- UniFew is a prompt-based model,  a class of models that tailor the input/output format of their data to match the format used during pretraining.
- While this tecunique allows them to perform a task without the need for additional classification layers, prompt-based models are **typically sensitive to the choice of the promts**.
- To avoid this issue, UniFew converts examples into multiple-choice question-answer(QA) format, and uses **UnifiedQA**, a **T5** model further pretrained on a large collection of QA pairs
> UnifiedQA : T5로 Text-2-Text QA Model 만듦. 4개의 QA Task를 모두 풀 수 있음.
- Two main strenghts:
  - First, the **prompt design problem is much simpler** because UnifiedQA questions had well-defined formats.
    - For example, we only need **four general prompt templates** which cover all 20 datsets in the FLEX benchmark, while prior works have needed specialized prompts for each dataset.
  - Second, UnifiedQA's multiple-choice format ensures the model outputs a valid class label, without the need for learned or manaully-defined mappings or verbalizers(말로 표현하다) required for other prompt-based methods.