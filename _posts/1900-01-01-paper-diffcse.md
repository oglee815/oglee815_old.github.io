---
layout: post
date: 2022-05-9 14:00
created_date: 2022-05-9 14:00
title: "[Paper] 2204 DiffCSE: Difference-based Contrastive Learning for Sentence Embeddings"
author: Yung-Sung Chuang, Yoon Kim et al.
description: "DiffCSE: Difference-based Contrastive Learning for Sentence Embeddings"
comments: true
math: true
category: 
- Paper
tags:
- NLP
- SimCSE
- DiffCSE
- Contrastive Learning
---

DiffCSE 리뷰
<!--more-->

# Abstract
_We propose **DiffCSE, an unsupervised contrastive learning framework** for learning sentence embeddings. DiffCSE learns sentence embeddings that are **sensitive to the difference between the original sentence and an edited sentence**, where the edited sentence is obtained by stochastically masking out the original sentence and then sampling from a masked language model. We show that DiffSCE is an instance of equivariant contrastive learning (Dangovski et al., 2021), which generalizes contrastive learning and learns representations that are insensitive to certain types of augmentations and sensitive to other “harmful” types of augmentations. Our experiments show that DiffCSE achieves state-of-the-art results among unsupervised sentence representation learning methods, outperforming unsupervised SimCSE1 by 2.3 absolute points on semantic textual similarity tasks_

# Introduction
- Contrastive learning uses **multiple augmentations** on a single datum to construct positive pairs whose representations are trained to be more similar to one another than negative pairs.
- **SimCSE** finds that constructing positive pairs via a **simple dropout-based augmentation** works much better than more complex augmentations such as word deletions or replacements based on synonyms or masked language models.
` This is because direct augmentations on the input often change the meaning of the sentence.`
- We propose to learn sentence representations that are **aware of, but not necessarily invariant to, such direct surface-level augmentations.**
- This is an **instance of equivariant contrastive learning**, which improves vision representation learning by using a contrastive loss on insensitive image transformations (e.g., grayscale) and a prediction loss on sensitive image transformations (e.g., rotations).
- We operationalize **equivariant contrastive learning on sentences** by using **dropout-based augmentation** as the insensitive transformation (as in SimCSE (Gao et al., 2021)) and **MLM-based word replacement** as the sensitive transformation. 
- We conduct experiments on **7 semantic textual similarity tasks (STS) and 7 transfer tasks from SentEval**
- DiffCSE approach can achieve around 2.3% absolute improvement over SimCSE.

# Related Work
## Equivariant(등변) Contrastive Learning
- **Invariant Contrastive Learning**
  - 입력 x에 대하여, f(T(x)) = f(x) 가 되도록 f를 훈련
  - Transformation에 invariant 한 값을 출력하도록 학습. **insensitive to Transformation(G)**
  - ![image](https://user-images.githubusercontent.com/18374514/167547477-16a66a22-2cf2-4c1b-b8a6-2fd90842fda4.png)
  - 그러나 Transformation에 따라서 Semantic 한 정보가 변경되는 경우가 허다하기 때문에 단점이 있는 학습법
  - Understanding the role of **input transformations is crucial** for successful contrastive learning.
  - But some of revealed transformations are shown to be harmful for CL.
    - Ex) image rotation or MLM in NLP
    - SimCSE에 나와 있는 MLM을 Augmentation 방식으로 사용했을 경우의 결과
    - ![image](https://user-images.githubusercontent.com/18374514/167335228-5ce32aea-97c9-4878-b397-57b60c5f627e.png)

- **Equivariant Contrastive Learning**
  - 입력 x에 대하여, f(T(x)) = T'(f(x)) 가 되도록 f를 훈련, T'이 identity function이면 ICL과 동일
  - T와 T'은 파라미터가 없는 fixed transformation
  - 따라서 저 식이 완성되려면 f가 T에 굉장히 Sensitive하게 학습이 되어야 함.
  - 이것을 **equivariance to Transformation(G)**라 표현함
`Insensitive, Sensitive는 그냥 있어 보이기 위한 표현 방식일 뿐인 것 같다는 생각이.. 사실 ICL에서도 f가 T에 상관없이 동일한 값을 내려면 T에 insensitive하기만 하면 되는건 아닌 것 같은데..`
  - equivariant CL 이미지 ![image](https://user-images.githubusercontent.com/18374514/167349581-e5424897-1fb5-4839-874c-bb0e3a57fd11.png)
  ![image](https://user-images.githubusercontent.com/18374514/167547516-4f5b93c6-6002-460e-9099-ed9d4b447b09.png)

# Difference-based Contrastive Learning
- DiffCSE는 결국 Equivariant CL 페이퍼의 NLP 버전이다. ECL에서처럼 Hybrid 방식으로 CL을 진행(ICL+ECL). 
- ![image](https://user-images.githubusercontent.com/18374514/167334643-80ad1e39-674b-463d-9114-8e4fcc71faa8.png)
- 위 그림의 왼쪽에서 보듯이, SimCSE의 Loss 또한 그대로 사용
- ![image](https://user-images.githubusercontent.com/18374514/167548059-cd176bbe-45f7-4aff-9e0b-5d799260c84d.png)
- **Sentence Encoder로는 BERT, RoBERTa** 등 사용
- 오른쪽은 **Conditional ELECTRA 사용**
  - input x에 random masking(30%)을 해서 X'을 만듦.
  - Mask 된 단어를 예측해서 새로운 X''을 만들어낼 Generator는 Fixed Pretrained LM사용
  - 원래 ELECTRA Paper에서처럼 Generator는 너무 MASK 단어를 잘 유추할 필요가 없으므로 **DistilBERT**를 사용
  - Discriminator는 X''의 각 token들이 Generator에 의해 replace 된건지 원래 Token인지 맞추는 binary classificaiton 문제를 학습(Replace Token Detection)
  - ![image](https://user-images.githubusercontent.com/18374514/167551606-0a466562-4478-4cb0-b200-205e05086214.png)
  - 그래서 Total Loss는
    - ![image](https://user-images.githubusercontent.com/18374514/167551864-73afc4c4-6f60-4200-9117-a3f2e135a271.png)
    - **최적 lambda는 0.005**
  - **근데 왜 Conditional 이냐?**
    - 왼쪽의 SentenceEncoder에서 넘어오는 **sentence representation h = f(x)**를 이용해서 RTD를 하기 때문
    - **이 **h** 가 RTD를 잘 할 수 있도록 충분히 informative 해 지도록 f가 학습이 되는 것이 중요**
    - This approach essentially makes the conditional discriminator perform a **“diff operation”,** hence the name DiffCSE.
- Genertor G는 Fix되고 SentenceEncoder f 와 Discriminator D는 학습됨. 
- 학습 후, **Sentence Embedding을 뽑아낼 때는 f만 사용**

# Experimants
- SimCSE와 대체적으로 비슷
- Sentence Embedding으로는 [CLS]를 쓰되, **MLP와 BatchNorm**을 통과시켜서 사용
  - ![image](https://user-images.githubusercontent.com/18374514/167557696-d010404c-9b0d-4f0c-aa5b-01abcd32a8db.png)

## Data
- Unsupervised Learning Training Data
  - Random 1M Wiki 문장
- Evaluation
  - 7 STS and 7 Transfer Task Dataset
 
## Results
- ![image](https://user-images.githubusercontent.com/18374514/167558318-f1ea357a-4573-4589-a3d8-36dca6616b14.png)
- ![image](https://user-images.githubusercontent.com/18374514/167558802-d983294b-2650-4560-8406-578031e65a37.png)

## Ablation Task
**1) Removing CL**
  -  CL 없이 RTD만 하게 되면, STS-B 점수는 30%가 떨어지지만 Transfer Task는 오직 2%만 하락. 
  - This result shows that it is important to have insensitive and sensitive attributes that **exist together in the representation space.**

**2) Next Sentence vs Same Sentence**
  - ELECTRA에는 Same Sentence가 아니라 Next Sentence를 넣음(이러면 Diff operation은 사라짐)
  - 그러나 효과는 구림.

**3) Other Conditional Pretraining Tasks**
  - RTD(Conditional binary difference prediction loss) 대신 Conditional MLM이나 Corrective LM을 사용했으나 성능은 구림
  - 구체적으로 어떻게 적용했느지에 대한 설명은 부족함

**4) Insert/Delete**
  - 원래 토큰을 다른 토큰으로 replace하는 방식 대신 다른 augmentation 방식 사용
  - Insert : 원래 문장 길이의 15% 의 token을 insert한 뒤(generator 이용), insert된 토큰인지 원래 있던 토큰인지 판별
  - Delete : 원래 토큰 중 15%를 지운 뒤, 남아 있는 각 토큰에 대하여 이어지는 토큰이 지워졌는지 아닌지 판별
  - 그러나 모두 결과는 좋지 않았음.
 
**5) Pooler Choice**
  - SimCSE는 BERT와 동일한 pooler(one linear layer with tanh activation function)를 사용했지만, 실험 해보니 2layer+batch norm이 더 좋았음.
  - 이건 Computer Vision쪽에서는 흔하게 사용되는 방식임.
  - **BatchNorm을 추가하는 방식은 SimCSE, DiffCSE 모두에서 좋았음.**

# Analysis
- SimCSE와 DiffCSE가 점수는 당연히 DiffCSE가 좋지만 정성적인 결과는 어떻게 다른지 비교
  - "You can do it, too."와 가장 유사한 문장을 2000여개 중에 찾아봐
    - SimCSE: "you can use it, too."
    - DiffCSE: "You can do it, too." --> 즉 미세한 차이도 분별해서 정확한 정답을 찾음.
  - ![image](https://user-images.githubusercontent.com/18374514/167563909-86557a8c-77a8-4cdb-a779-fd2584fd5dfd.png)
- 가장 유사한 문장을 찾는 Retrieval Task에서도 DiffCSE가 압도하는 성능을 보임
  - ![image](https://user-images.githubusercontent.com/18374514/167564226-e72cd2e2-7589-48d2-b01b-40ff827b1fa8.png)
-  Distribution of Sentence Embedding
  - ![image](https://user-images.githubusercontent.com/18374514/167564325-ff440145-e9e4-4450-9fdd-cff31b702537.png)
  - Transformer base PLM의 경우 representation space를 squeeze하는 현상이 있는데, 위 그림에서도 DiffCSE가 내놓는 점수들이 좀 몰려있는 현상이 있음.
  - 따라서 uniformity and alignment 를 체크해본 결과, SimCSE는 uniformity에서 장점이 있지만 DiffCSE는 alignment에서 장점이 있음.
  - It indicates that SimCSE and DiffCSE are optimizing the representation space in two different directions. And **the improvement of DiffCSE may come from its better alignment.**
  `Uniformity는 단지 전체 data의 representation이 얼마나 펼쳐져 있는지를 보는 거고, Alignment는 positve pair끼리 얼마나 가까이 뭉쳐여 있는지를 보는 것이기 때문에 alignment가 더 어떤 task를 수행하는데 도움이 되지 않을까 하는 개인적인 생각`

# Conclusion
- DiffCSE 제안, unsupervised sentence embedding frame work, aware of, but not invariant to MLM based-word replacement 
- 여러 결과에서 SimCSE보다 좋은 결과를 보임

# Take Home Messages 🤔
- Invariant, Equivariant CL에 대한 이해 중요
- Conditional ELECTRA 개념이 매우 흥미로웠음
- DiffCSE가 성능이 좋긴한데, 학습할때 3가지 모델을 메모리에 띄어놓고 학습해야하므로 비효율적인듯..? STS 결과는 DCPCSE랑 비슷하지만 훨씬 메모리가 많이 먹는다.
- Pooler에 BatchNorm을 쓰는거는 굿굿

# Reference
- DiffCSE : https://arxiv.org/abs/2204.10298
- Equivariant CL : https://arxiv.org/abs/2111.00899
- SimCSE : https://arxiv.org/abs/2104.08821
