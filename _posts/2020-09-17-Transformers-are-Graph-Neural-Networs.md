---
layout: post
date: 2020-09-17 14:08
reviewed_date: 2020-09-17 14:08
title: Transformers are Graph neural Networks
description: Transformers are Graph neural Networks
comments: true
math: true
category: 
- NLP
tags:
- Transformers
- GNN
---

GNN 덕분에 Transformer 에서 Attention을 없애거나 개선 할 수 있을까?
<a href='https://thegradient.pub/transformers-are-graph-neural-networks/'>[원문]</a>

 <!--more-->

### Intro
Graph Neural Networks(GNN)이 Pinterest 나 Alibaba, Twitter 등의 추천 시스템에서 쓰이고는 있지만 사실 더욱 영리하게 쓰이는 부분은 **Transformer Architecture** 에서 이다. GNN 과 NLP(Transformer) 사이의 관계에 대해서 알아보자.

## Representation Learning for NLP
결국 모든 NN은 데이터의 Latent or hidden representation을 잘 학습해서 Classification이나 Translation 과 같은 Generation Task에서 좋은 성능을 내야한다. 이때 이용하는 것이 일반적으로 error/loss function이다.

전통적으로는 RNN이 NLP에서는 많이 쓰였다. (저자가 추천하는 LSTM 설명 링크 : http://colah.github.io/posts/2015-08-Understanding-LSTMs/)
Transformer가 등장하면서 RNN은 대부분 대체되었다. Transformer는 recurrence한 방법 대신에 attention mechanism을 이용하여 representation을 구축한다.
one-word-at-a-time 스타일의 RNN을 Attention이 대체를 하게 된다. (attention is all you need: https://www.youtube.com/watch?v=iDulhoQ2pro)

## Breaking down the Transformer
We update the hidden feature $h$ of the $i$'th word in a sentence  from layer  to layer  as follows:

\\[h_{i}^{\ell+1} = \text{Attention} \left( Q^{\ell} h_{i}^{\ell} \ , K^{\ell} h_{j}^{\ell} \ , V^{\ell} h_{j}^{\ell} \right) \\]

i.e.,

\\[ h_{i}^{\ell+1} = \sum_{j \in \mathcal{S}} w_{ij} \left( V^{\ell} h_{j}^{\ell} \right)\\]

\\[\text{where} \ w_{ij} = \text{softmax}_j \left ( Q^{\ell} h^{\ell}_i \cdot  K^{\ell} h^{\ell}_j  \right)\\]

where \\(j \in \mathcal{S}\\) denotes the set of words in the sentence and \\(Q^{\ell}, K^{\ell}, V^{\ell}\\) are learnable linear weights (denoting the **Q**uery, **K**ey and **V**alue for the attention computation, respectively).

RNN 과 차별화 되는 장점 한가지, 
The attention mechanism is performed parallelly for each word in the sentence to obtain their updated features in **one shot**—another plus point for Transformers over RNNs, which update features word-by-word.

![Attention Architecture](/assets/img/attention-block.jpg){:height="50%" width="50%"}

## Multi-head Attention
Getting this straightforward dot-product attention mechanism to work proves to be tricky. **Bad random initializations of the learnable weights can de-stabilize the training process.(정확히 무슨 의미 일까?)**
We can overcome this by parallelly performing multiple 'heads' of attention and concatenating the result (with each head now having separate learnable weights):


\\[h_{i}^{\ell+1} = \text{Concat} \left( \text{head}_1, \ldots, \text{head}_K \right) O^{\ell},\\]

\\[\text{head}_k = \text{Attention} \left(  Q^{k,\ell} h_i^{\ell} \ , K^{k, \ell} h_j^{\ell} \ , V^{k, \ell} h_j^{\ell} \right),\\]

where \\(Q^{k,\ell}, K^{k,\ell}, V^{k,\ell}$\\) are the learnable weights of the \\(k\\)'th attention head and \\(O^{\ell}\\) is a down-projection to match the dimensions of \\(h_i^{\ell+1}\\) and \\(h_i^{\ell}\\) across layers.

Multiple heads allow the attention mechanism to essentially 'hedge its bets'(분산 투자 하는 느낌?), looking at **different transformations or aspects of the hidden features from the previous layer.** We'll talk more about this later.

## Scaling Issues
A key issue motivating the final Transformer architecture is that the features for words after the attention mechanism might be at **different scales or magnitudes.** This can be due to **some words having very sharp or very distributed attention weights** when summing over the features of the other words. Additionally, at the individual feature/vector entries level, concatenating across multiple attention heads—each of which might output values at different scales—can lead to the entries of the final vector \\(h_{i}^{\ell+1}\\) having a wide range of values.
Following conventional ML wisdom, it seems reasonable to add a **normalization layer** into the pipeline.

Transformers overcome issue (2) with **LayerNorm**, which normalizes and learns an affine transformation at the feature level. Additionally, **scaling the dot-product** attention by the square-root of the feature dimension helps counteract issue (1).
> attention map(Q*K)를 root(dimension) 으로 나눔

Finally, the authors propose another 'trick' to control the scale issue: **a position-wise 2-layer MLP** with a special structure. After the multi-head attention, they project \\(h_i^{\ell+1}\\) to a (absurdly) higher dimension by a learnable weight, where it undergoes the ReLU non-linearity, and is then projected back to its original dimension followed by another normalization:

\\[h_i^{\ell+1} = \text{LN} \left( \text{MLP} \left( \text{LN} \left( h_i^{\ell+1} \right) \right) \right)\\]
> HuggingFace 코드에서 BertSelfOutput, BertIntermediate 을 보면 될 듯?

To be honest, **I'm not sure what the exact intuition behind the over-parameterized feed-forward sub-layer was.** I suppose LayerNorm and scaled dot-products didn't completely solve the issues highlighted, so the big MLP is a sort of hack to re-scale the feature vectors independently of each other. According to Jannes Muenchmeyer, the feed-forward sub-layer ensures that the Transformer is a universal approximator. Thus, projecting to a very high dimensional space, applying a non-linearity, and re-projecting to the original dimension allows the model to **represent more functions than maintaining the same dimension across the hidden layer would.**
> BERT Base 기준으로 768 차원의 벡터가 3024로 늘어났다가 다시 768로 줄어드는데, 이 부분에 대한 해석이지만 명확하지는 않다.

![Tansformer Architecture](/assets/img/transformer-block.png){:height="50%" width="50%"}

The Transformer architecture is also extremely **amenable** to very deep networks, enabling the NLP community to **scale up** in terms of both model parameters and, by extension, data.
**Residual connections** between the inputs and outputs of each multi-head attention sub-layer and the feed-forward sub-layer are key for stacking Transformer layers (but omitted from the diagram for clarity).

## GNNs build representations of graphs
**Graph Neural Networks** (GNNs) or **Graph Convolutional Networks** (GCNs) build representations of nodes and edges in graph data. They do so through neighbourhood aggregation (or message passing), where each node gathers features from its neighbours to update its representation of the local graph structure around it. Stacking several GNN layers enables the model to propagate(전파) each node's features over the entire graph—from its neighbours to the neighbours' neighbours, and so on.

![Gnn Network](/assets/img/gnn-social-network.jpg){:height="50%" width="50%"}

In their most basic form, GNNs update the hidden features \\(h\\) of node \\(i\\) (for example, 😆) at layer \\(\ell\\) via a non-linear transformation of the node's own features \\(h_i^{\ell}\\) added to the aggregation of features \\(h_j^{\ell}\\) from each neighbouring node \\(j \in \mathcal{N}(i)\\): 

\\(h_{i}^{\ell+1} =  \sigma \Big( U^{\ell} h_{i}^{\ell} + \sum_{j \in \mathcal{N}(i)} \left( V^{\ell} h_{j}^{\ell} \right)  \Big),\\)

where \\(U^{\ell}, V^{\ell}\\) are learnable weight matrices of the GNN layer and \\(\sigma\\) is a non-linear function such as ReLU. In the example, \\(\mathcal{N}\\)(😆)  { 😘, 😎, 😜, 🤩 }.

The summation over the neighbourhood nodes \\(j \in \mathcal{N}(i)\\)can be replaced by other input size-invariant **aggregation** functions such as simple mean/max or something more powerful, such as a weighted sum via an **attention mechanism**.

Does that sound familiar? Maybe a pipeline will help make the connection:

![Gnn block](/assets/img/gnn-block.jpg){:height="50%" width="50%"}

If we were to do multiple parallel heads of neighbourhood aggregation and replace summation over the neighbours  with the attention mechanism, i.e., a weighted sum, we'd get the **Graph Attention Network (GAT)**. Add normalization and the feed-forward MLP, and voila, we have a **Graph Transformer!**

## Sentences are fully-connected word graphs
To make the connection more explicit, consider a sentence as a fully-connected graph, where each word is connected to every other word. Now, we can use a GNN to build features for each node (word) in the graph (sentence), which we can then perform NLP tasks with.
![Tansformer Architecture](/assets/img/gnn-nlp.jpg)

Broadly, this is what Transformers are doing: they are GNNs with multi-head attention as the neighbourhood aggregation function. Whereas standard GNNs aggregate features from their local neighbourhood nodes \\(j \in \mathcal{N}(i)\\), Transformers for NLP treat the entire sentence \\(\mathcal{S}\\) as the local neighbourhood, aggregating features from each word \\(j \in \mathcal{S}\\) at each layer.

Importantly, various problem-specific tricks—such as position encodings, causal/masked aggregation, learning rate schedules and extensive pre-training—are essential for the success of Transformers but seldom seem in the GNN community. At the same time, looking at Transformers from a GNN perspective could inspire us to get rid of a lot of the bells and whistles in the architecture.

## Lessons
### Are sentences fully connected graphs

Now that we've established a connection between Transformers and GNNs, let me throw some ideas around. For one, **are fully-connected graphs the best input format for NLP?**

Before statistical NLP and ML, linguists like Noam Chomsky focused on developing formal theories of linguistic structure, such as syntax trees/graphs. Tree LSTMs already tried this, but maybe Transformers/GNNs are better architectures for bringing together the two worlds of linguistic theory and statistical NLP? For example, a very recent work from MILA and Stanford explores augmenting pre-trained Transformers such as BERT with syntax trees <ins>[Sachan et al., 2020](https://arxiv.org/abs/2008.09084). </ins>
![Tansformer Architecture](/assets/img/syntax-tree.png)

### Long term dependencies
Another issue with fully-connected graphs is that they make **learning very long-term dependencies between words difficult.** This is simply due to how the number of edges in the graph scales quadratically with the number of nodes, i.e., in an $n$ word sentence, a Transformer/GNN would be doing computations over $n^2$ pairs of words. Things get out of hand for very large $n$.

The NLP community's perspective on the long sequences and dependencies problem is interesting: **making the attention mechanism** <ins>**[sparse](https://openai.com/blog/sparse-transformer/)**</ins> or <ins>**[adaptive](https://ai.facebook.com/blog/making-transformer-networks-simpler-and-more-efficient/)**</ins> in terms of input size, adding <ins>**[recurrence](https://ai.googleblog.com/2019/01/transformer-xl-unleashing-potential-of.html)**</ins> or compression into each layer, and using Locality Sensitive Hashing for efficient attention are all promising new ideas for better transformers. See Maddison May's excellent survey on long-term context in Transformers for more details.

It would be interesting to see ideas from the GNN community thrown into the mix, e.g., Binary Partitioning for sentence graph sparsification seems like another exciting approach. BP-Transformers recursively sub-divide sentences into two until they can construct a hierarchical binary tree from the sentence tokens. This structural inductive bias helps the model process longer text sequences in a memory-efficient manner.

It would be interesting to see ideas from the GNN community thrown into the mix, e.g., **Binary Partitioning** for sentence **graph sparsification** seems like another exciting approach. BP-Transformers recursively sub-divide sentences into two until they can construct a hierarchical binary tree from the sentence tokens. This structural inductive bias helps the model process longer text sequences in a memory-efficient manner.
![Tansformer Architecture](/assets/img/long-term-depend.png)

### Are Transformers learning neural syntax?
There have been several interesting papers from the NLP community on what Transformers might be learning. The basic premise is that performing attention on all word pairs in a sentence—with the purpose of identifying which pairs are the most interesting—enables Transformers to learn something like a **task-specific syntax.** **Different heads** in the multi-head attention might also be 'looking' at **different syntactic properties.**
![Tansformer Architecture](/assets/img/attention-heads.png)

### Why multiple heads of attention? Why attention?
I'm more sympathetic to the optimization view of the multi-head mechanism—having multiple attention heads improves learning and overcomes bad random initializations. For instance, these papers showed that Transformer heads can be 'pruned' or removed after training without significant performance impact.

**Multi-head neighbourhood aggregation mechanisms** have also proven effective in GNNs, e.g., **GAT** uses the same multi-head attention, and **MoNet** uses multiple Gaussian kernels for aggregating features. Although invented to stabilize attention mechanisms, could the multi-head trick become standard for squeezing out extra model performance?

Conversely, GNNs with simpler aggregation functions such as sum or max do not require multiple aggregation heads for stable training. **Wouldn't it be nice for Transformers if we didn't have to compute pair-wise compatibilities between each word pair in the sentence?**

Could Transformers benefit from ditching(버리다) attention, altogether? Yann Dauphin and collaborators' recent work suggests an alternative ConvNet architecture. **Transformers, too, might ultimately be doing something similar to ConvNets!**
![Tansformer Architecture](/assets/img/attention-conv.png)

### Why is training Transformers so hard?
Reading new Transformer papers makes me feel that training these models requires something akin to black magic when determining the best learning rate schedule, warmup strategy and decay settings. This could simply be because the models are so huge and the NLP tasks studied are so challenging.

But recent results suggest that it could also be due to the specific permutation of normalization and residual connections within the architecture.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I enjoyed reading the new <a href="https://twitter.com/DeepMind?ref_src=twsrc%5Etfw">@DeepMind</a> Transformer paper, but why is training these models such dark magic? &quot;For word-based LM we used 16, 000 warmup steps with 500, 000 decay steps and sacrifice 9,000 goats.&quot;<a href="https://t.co/dP49GTa4ze">https://t.co/dP49GTa4ze</a> <a href="https://t.co/1K3Fx4s3M8">pic.twitter.com/1K3Fx4s3M8</a></p>&mdash; Chaitanya Joshi (@chaitjo) <a href="https://twitter.com/chaitjo/status/1229335421806501888?ref_src=twsrc%5Etfw">February 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I know I'm ranting, but this makes me skeptical: **Do we really need multiple heads of expensive pair-wise attention, overparameterized MLP sub-layers, and complicated learning schedules?** Do we really need **massive models** with massive carbon footprints? Shouldn't architectures with good <ins>**[inductive biases](https://arxiv.org/abs/1806.01261)**</ins> for the task at hand be easier to train?