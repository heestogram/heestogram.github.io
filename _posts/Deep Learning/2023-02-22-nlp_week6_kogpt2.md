---
title: "[NLP] KoGPT2로 이어질 문장을 예측해보자"
excerpt: "문장 생성에 특화된 KoGPT2로 한국어 문장을 생성해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, KoGPT2, pytorch, keras]

published: true

categories:
  - DL

date: 2023-02-22 21:00:00
last_modified_at: 2023-02-22 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

GPT-2는 주어진 텍스트의 다음 단어를 잘 예측할 수 있도록 학습된 언어모델이며 문장 생성에 최적화 되어 있다. KoGPT-2는 이름에서 알 수 있듯, 한국어의 언어적 한계를 해결하기 위한 한국어 전용 언어모델이다.

BERT가 트랜스포머에서 디코더를 지우고 인코더만 남겼던 것과 완전히 반대로, GPT-2는 디코더만 남긴다. 또한 BERT와 달리 GPT-2는 한 번에 하나의 토큰을 출력하고 출력 토큰을 다음 단계의 입력으로 사용한다. 즉, 과거 언어모델과는 유사한 구조를 띤다.

<br>

## KoGPT2로 이어지는 문장 생성


```python
!pip install transformers
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting transformers
      Downloading transformers-4.26.1-py3-none-any.whl (6.3 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m6.3/6.3 MB[0m [31m38.3 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pyyaml>=5.1 in /usr/local/lib/python3.8/dist-packages (from transformers) (6.0)
    Requirement already satisfied: tqdm>=4.27 in /usr/local/lib/python3.8/dist-packages (from transformers) (4.64.1)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers) (2.25.1)
    Requirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers) (3.9.0)
    Collecting huggingface-hub<1.0,>=0.11.0
      Downloading huggingface_hub-0.12.0-py3-none-any.whl (190 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m190.3/190.3 KB[0m [31m10.9 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: packaging>=20.0 in /usr/local/lib/python3.8/dist-packages (from transformers) (23.0)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (2022.6.2)
    Collecting tokenizers!=0.11.3,<0.14,>=0.11.1
      Downloading tokenizers-0.13.2-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (7.6 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m7.6/7.6 MB[0m [31m56.8 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (1.21.6)
    Requirement already satisfied: typing-extensions>=3.7.4.3 in /usr/local/lib/python3.8/dist-packages (from huggingface-hub<1.0,>=0.11.0->transformers) (4.4.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2.10)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (1.24.3)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2022.12.7)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (4.0.0)
    Installing collected packages: tokenizers, huggingface-hub, transformers
    Successfully installed huggingface-hub-0.12.0 tokenizers-0.13.2 transformers-4.26.1
    

huggingface에 배포된 모델들은 Pytorch로 학습된 모델이기 때문이다. 만약 Tensorflow에서 사용하고 싶다면 from_pretrained() 인자에 `from_pt=True`를 넣어줌으로써 Tensorflow모델로 로드할 수 있다.

pytorch와 tensorflow로 모두 진행해보겠다.


```python
import torch
from transformers import PreTrainedTokenizerFast
from transformers import GPT2LMHeadModel

tokenizer = PreTrainedTokenizerFast.from_pretrained('skt/kogpt2-base-v2')
model = GPT2LMHeadModel.from_pretrained('skt/kogpt2-base-v2')
```

    The tokenizer class you load from this checkpoint is not the same type as the class this function is called from. It may result in unexpected tokenization. 
    The tokenizer class you load from this checkpoint is 'GPT2Tokenizer'. 
    The class this function is called from is 'PreTrainedTokenizerFast'.
    


```python
text = '수강신청에서 원하는 강의를 신청하기 위해서는'

input_ids = tokenizer.encode(text, return_tensors='pt')
summary_ids = model.generate(input_ids,
                           max_length=128,
                           repetition_penalty=2.0,
                           use_cache=True)
```


```python
decoding_ids = tokenizer.decode(summary_ids[0])
print(decoding_ids)
```

    수강신청에서 원하는 강의를 신청하기 위해서는 반드시 해당 강의에 대한 사전정보를 확인해야 한다.
    또한 각 강좌별 전문강사진의 상세한 설명과 함께 질의응답 시간도 마련돼 있다.
    이번 과정은 오는 9월 30일까지 매주 화요일 오후 2시부터 4시까지 총 6회에 걸쳐 진행된다.
    수강을 희망하는 사람은 누구나 무료로 참여할 수 있으며, 자세한 사항은 홈페이지(www.suwonline.kr)를 통해 확인할 수도 있다.</d> 지난해 12월 31일부터 올 1월 1일 사이에 발생한 교통사고 사망자 수는 모두 1만2천9백여 명으로 집계됐습니다.
    이는 전년 같은 기간보다 8% 증가한 수치입니다.
    특히, 전체 사고
    


```python
print(input_ids)
print(summary_ids)
```

    tensor([[32619, 10309,  9023, 15605, 25175, 48565, 11357]])
    tensor([[32619, 10309,  9023, 15605, 25175, 48565, 11357, 11488,  9992, 11631,
              8022,  9167, 14827, 32081, 10190,  9685,  9178,  7335,  8704,  9188,
              9120,  8224,  7644, 10161,  6835,  7756, 14900, 27390, 10250,  6903,
              9304, 37989,  8142,  7192, 10135,  7235, 10488,  7249,  9029,  8146,
              7623, 21579, 14472, 10151, 15179,  9168, 40352,  9192, 25483, 21734,
              9045, 49036,  9130, 38916,  9390,  9191, 11687, 10219, 32800,  7847,
             14259, 43728, 11146, 18818, 41730, 30748,  9025,  9418, 20616, 27146,
             46483,  9982, 43804,   389,   457,   459,   461,  9549,   450, 14605,
             43832,  9291,  9430, 15695,  9788, 10960,     8, 12199,  8711, 10033,
             13805,  9148,  9456, 10093, 10087, 10113, 14114, 10971, 27011, 11317,
              8159, 11688,  9432, 12283,   393,  8361,   400,  7610,  8030, 21442,
             30112,  7250, 10010,  9489,  9034,  7109,  9239, 10808,  9518,  9253,
               380, 41910, 27035, 10542, 26421,   387,  9759, 12329]])
    


```python
import tensorflow as tf
from transformers import AutoTokenizer
from transformers import TFGPT2LMHeadModel

tokenizer = AutoTokenizer.from_pretrained('skt/kogpt2-base-v2')
model = TFGPT2LMHeadModel.from_pretrained('skt/kogpt2-base-v2', from_pt=True)
```

    Special tokens have been added in the vocabulary, make sure the associated word embeddings are fine-tuned or trained.
    Some weights of the PyTorch model were not used when initializing the TF 2.0 model TFGPT2LMHeadModel: ['transformer.h.2.attn.masked_bias', 'transformer.h.7.attn.masked_bias', 'transformer.h.9.attn.masked_bias', 'lm_head.weight', 'transformer.h.0.attn.masked_bias', 'transformer.h.11.attn.masked_bias', 'transformer.h.6.attn.masked_bias', 'transformer.h.4.attn.masked_bias', 'transformer.h.5.attn.masked_bias', 'transformer.h.1.attn.masked_bias', 'transformer.h.10.attn.masked_bias', 'transformer.h.3.attn.masked_bias', 'transformer.h.8.attn.masked_bias']
    - This IS expected if you are initializing TFGPT2LMHeadModel from a PyTorch model trained on another task or with another architecture (e.g. initializing a TFBertForSequenceClassification model from a BertForPreTraining model).
    - This IS NOT expected if you are initializing TFGPT2LMHeadModel from a PyTorch model that you expect to be exactly identical (e.g. initializing a TFBertForSequenceClassification model from a BertForSequenceClassification model).
    All the weights of TFGPT2LMHeadModel were initialized from the PyTorch model.
    If your task is similar to the task the model of the checkpoint was trained on, you can already use TFGPT2LMHeadModel for predictions without further training.
    


```python
text = '수강신청에서 원하는 강의를 신청하기 위해서는'

input_ids = tokenizer.encode(text)
input_ids = tf.convert_to_tensor([input_ids])

summary_ids = model.generate(input_ids,
                        max_length=128,
                        repetition_penalty=2.0,
                        use_cache=True)
output_ids = summary_ids.numpy().tolist()[0]
decoding_ids = tokenizer.decode(output_ids)
print(decoding_ids)

#tensorflow가 pytorch보다 런타임이 좀 더 길었다.
```

    수강신청에서 원하는 강의를 신청하기 위해서는 반드시 해당 강의에 대한 사전정보를 확인해야 한다.
    또한 각 강좌별 전문강사진의 상세한 설명과 함께 질의응답 시간도 마련돼 있다.
    이번 과정은 오는 9월 30일까지 매주 화요일 오후 2시부터 4시까지 총 6회에 걸쳐 진행된다.
    수강을 희망하는 사람은 누구나 무료로 참여할 수 있으며, 자세한 사항은 홈페이지(www.suwonline.kr)를 통해 확인할 수도 있다.</d> 지난해 12월 31일부터 올 1월 1일 사이에 발생한 교통사고 사망자 수는 모두 1만2천9백여 명으로 집계됐습니다.
    이는 전년 같은 기간보다 8% 증가한 수치입니다.
    특히, 전체 사고
    

<br>

## top5 뽑아서 문장 생성

https://huggingface.co/blog/how-to-generate 링크를 참고해보니 top-n-sampling의 아이디어를 따온 것 같다.


```python
import numpy as np
output = model(input_ids)
output.logits
```




    <tf.Tensor: shape=(1, 7, 51200), dtype=float32, numpy=
    array([[[-6.7055902, -6.0299535, -5.8297663, ..., -4.3834677,
             -3.7205324, -2.8471978],
            [-6.0724792, -6.502076 , -5.002329 , ..., -4.7735524,
             -5.1650953, -5.800695 ],
            [-5.820129 , -5.20039  , -4.6331215, ..., -1.8739362,
             -1.662243 , -2.7418447],
            ...,
            [-5.6954937, -5.7553587, -5.564661 , ..., -2.0856616,
             -4.215129 , -2.226934 ],
            [-5.221056 , -4.9581237, -6.161088 , ..., -3.1471634,
             -4.291755 , -6.088346 ],
            [-6.5182123, -6.252165 , -6.3499517, ..., -0.6392769,
             -4.6847677, -2.7679315]]], dtype=float32)>



가장 마지막 layer의 logit값들 중 top5의 값들만 따온다.


```python
top5 = tf.math.top_k(output.logits[0, -1], k=5)
```


```python
tokenizer.convert_ids_to_tokens(top5.indices.numpy())
```




    ['▁반드시', '▁해당', '▁수강', '▁먼저', '▁사전에']



확률분포에서 가장 가능성이 높은 5개를 샘플링하고 이들 중 랜덤으로 하나를 다음 단어로 예측하는 과정을 계속 반복한다.


```python
text = "수강신청에서 원하는 강의를 신청하기 위해서는"
input_ids = tokenizer.encode(text)
```


```python
import random

while len(input_ids) < 50:
    output = model(np.array([input_ids]))
    top5 = tf.math.top_k(output.logits[0, -1], k=5)
    token_id = random.choice(top5.indices.numpy())
    input_ids.append(token_id)
```


```python
tokenizer.decode(input_ids)
```




    '수강신청에서 원하는 강의를 신청하기 위해서는 사전에 수강신청서 등을 구비해 신청서를 작성, 제출해야 한다.\n신청을 원하는 학생은 다음주 월, 화요일에 신청을 할수 있다.\n수강은 무료이고 선정은 오는 9월10일까지며 자세한 내용은 교육청'


