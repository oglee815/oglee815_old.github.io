---
layout: post
date: 2022-08-11 00:00
created_date: 2022-08-11 00:00
title: "[study] text-to-image"
author: oglee
description: text-to-image 관련 페이퍼 정리
comments: true
math: true
category:
- Study
tags:
- text-to-image
- paper
- dalle
- imagen
- parti
- glide
---

text-to-image 관련 페이퍼 정리, dall-e, imagen, parti, glide
<!--more-->

# 2101 dall-e
## Summary
- Made by OpenAI
- DALL·E is a 12-billion parameter version of **GPT-3** trained to generate images from text descriptions, using a dataset of **text–image pairs**.
- MS-COCO dataset, without using any of the training labels.
- Image-to-Image translation at a rudimentary(가장 기본적인) level

## Method
- Our goal is to train a transfrormer to autoregressively model **the text and image tokens** as a single stream of data.
- Using pixels directly as image tokens would **require an inordinate(지나친) amount of memory** for high-resolution images(MNIST만 해도 28*28 = 784).
  - Stage 1: Train a discrete variational autoencoder(dVAE) to compress each 256x256 RGB image into a 32x32 grid of image tokens, each element of which can assume 8192 possible values.
  - Stage 2: Concatenate up to 256 BPE-encoded text tokens with 32x32=1024 image tokens, and train an autoregressive transformer to model the joint distribution over the text and image tokens
- 
