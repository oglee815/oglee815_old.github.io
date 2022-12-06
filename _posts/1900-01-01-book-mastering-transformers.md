---
layout: post
date: 2022-11-23 00:00
created_date: 2022-11-23 00:00
title: "[Book] Mastering Transformers"
author: Savas Yildirim
description: "Mastering Transformers"
comments: true
mathjax: true
category:
- book
tags:
- NLP
- Transfomers
---

Mastering Transformers 
<!--more-->

### Self-attention mechanism
- ![image](https://user-images.githubusercontent.com/18374514/203566128-2471f22b-3f6d-4ea9-a349-b24b084f5f23.png)

### Multi-head attention 
- ![image](https://user-images.githubusercontent.com/18374514/203567247-bc29a6fe-7786-446b-aaf7-e58464549cba.png)
- ![Dp8ERiD](https://user-images.githubusercontent.com/18374514/203567844-ab58e767-da31-4759-8c70-b1a96fec4843.png)

### Simple Distilation code
- ![image](https://user-images.githubusercontent.com/18374514/203574281-7f698ccb-a31b-4c59-a628-46af1a79c925.png) from [link](https://medium.com/huggingface/distilbert-8cf3380435b5)

### How to prune the model
    from torch.nn.utils import prune
    pruner = prune.L1Unstructured(amount=0.2)
    state_dict = distilroberta.state_dict()
    for key in state_dict.keys():
        if "weight" in key:
            state_dict[key] = pruner.prune(state_dict[key])
    distilroberta.load_state_dict(state_dict)

### Quantization
- FP16, Mixed Precision등을 쓰는것
- float 대신 int8로 quantization 하는 법
```
import torch
distilroberta = torch.quantization.quantize_dynamic(
    model = distilroberta,
    qconfig_spec ={torch.nn.Linear : torch.quantization.default_dynamic_qconfig},
    dtye=torch.qint8
    )
```
### Sparse attention with fixed patterns
- ![image](https://user-images.githubusercontent.com/18374514/203922841-faa7f1c5-8fc6-4075-bdfa-2443a4f0b58f.png)
- In sparse mode, the information is **transmitted through connected nodes(non-empty cells)** in the model.
  - For example, the output position 7 of the sparse attention matrix cannot directly attend to the input position 3, since the cell(7,3) is seen as empty.
  - However, position 7 indirectly attends to position 3 via the token position 5, that is (7->5, 5->3, 즉, 7->3이 가능)
  - the full self-attention incurs n^2 numbers of active cells, the sparse model does roughly **5xn**.
- Global Attention
  - ![image](https://user-images.githubusercontent.com/18374514/203923854-044f8d66-f0a8-4fd0-8acd-586cb9a22777.png)
  - A few selected tokens or a few injected tokens are used as global attention that can attend to all other positions and be attended by them.
  - Hense, the maximum path distance between any **two token positions is equal to 2.**
  - `그니까 최소한 맨 앞에 있는 CLS Token은 모든 token과 direct conntect가 가능하도록 설계된거네. SequenceClassification은 이것만으로도 잘될수도 있겠다. 그런데 Token Classification은?`
  - By the way, these global tokens don't have to be at the beginning of the sentence either. For example, the **longformer model randomly selects global tokens in addition to the first two tokens.**
  - ![image](https://user-images.githubusercontent.com/18374514/203927779-97c0e2d7-b4cc-42d8-9306-23e9e4c66cb6.png)
  - Blockwise Pattern, it chunkes the tokens into a fixed number of blocks, which is especially usueful for long-context-problems. For example, when a 4096 x 4096 attention matrix is chunked using a block size of 512, then 8 (512x512) query blocks and key blocks are formed.
  - Many efficient models such as **BigBird and Reformer mostly chunk tokens into blocks to reduce the complexity.**
- It is import to note that proposed patters must be supprted by **the accelerators and the libraries**. 
- Some attention patterns such as dilated patterns require a special matrix multiplication that is not directly supported in current deep learning libraries such as PyTorch or TensorFlow as of writing this chapter.
- Longformer uses a combination of a sliding window and global attention.

### Learnable patterns
- Learning-based patterns are the alternatives to fixed (predefined) patterns. 
- These approaches extract the patterns in an unsupervised data-driven fashion.
- They leverage some techniques to measure the **similarity between the queries and the keys to properly cluster them**
- This transformer family learns first **how to cluster the tokens** and then **restrict the interaction to get an optimum view of the attention matrix.**
- **Reformer**, as one of the important efficient models based on learnable patterns.
  - It employs **Local Self Attention (LSA) that cuts the input into N chnunks to reduce the complexity bottleneck.**
  - The most important contribution of Reformer is to leverage the **Locality Sensitive Hashing (LSH) function**, which **assignes the same value to similar query vectors**. Attention could be approximated by only **comparing the most similar vectors**, which helps us reduce the dimensionality and then sparsify the matrix. It is a safe operation since the softmax function is highly dominated by large values and can ignore dissimilar vectors. Additionally, insted of finding the relevant keys to a given query, only similar queries are found and bucked. That is, the position of a query can only attend to the positions of other queries to which it has a high cosine similarity.
  - To reduce the memory footprint, Reformer uses **reversible residual layers**, which **avoids the need to store the activations of all the layers to be reused for backpropagation**, following **Reversible Residual Network(RevNet)**, becuase the activations of any layer can be recovered from the activation of the following layer.
- It is important to note that Reformer model and many other efficient transformers are criticized as, in practice, **they are only more efficient than the vanilla transformer when the input length is very long**(Ref:Efficient Transformers:A Survey).

### Low-rank factorication, kernel methods, and other approaches
- The latest trend of the efficient model is to leverage low-rank approximations of the full self-attention matrix. 
  - These models are considered to be the lightest since they can reduce the self-attention complexity O(n2) to O(n) in both computational time and memory footprint. 
  - Choosing a very small projection dimension k, such that k << n, then the memory and space complexity is highly reduced. 
  - **Linformer and Synthesizer** are the models that efficiently approximate the full attention with a low-rank factorization. The decompose the dot-product **NxN attention of the original transformer through linear projections.**
- Kernel attention
  - **A kernel is a function that takes two vectors as arguments and returns the product of their projection with a feature map.**
  - It enables us to operate in high-dimensional feature space without even computing the coordinate of the data in that high-dimensional space, because computations within that space become more expensive. **This is when the kernel trick comes into play.**
  - The efficient models based on kernelization enables us to **re-write the self-attention mechanism to avoid explicitly computing the NxN matrix**.
  - **SVM하고 비슷한듯**
  - **Performer** and **Linear Transformers**
- 
