---
title: "[NLP] BERT 학습 방식 MLM과 NSP 연습"
excerpt: "BERT의 pretrain 방식인 MLM(Masked LM)과 NSP(Nest Sentence Prediction)을 구현해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, BERT, MLM, NSP]

published: true

categories:
  - DL

date: 2023-02-28 21:00:00
last_modified_at: 2023-02-28 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

## 1. 개요
BERT는 레이블이 없는 데이터로 pretrain시키고, 이후에 레이블이 있는 데이터로 추가 학습(fine tuning)을 하여 최적의 파라미터를 찾는 모델이다.

pretrain의 종류는 크게 두 가지가 있다.

1) **Mask LM**: 입력 단어의 15%를 masking하고 이를 예측하게 함.

  EX) 나는 운좋게도 수강신청에서 [MASK] 을 할 수 있었다. ⇒ [올클]

  이 때 15%를 모두 MASK 처리하는 것이 아니다.

  - 12%: [MASK]로 변경
  - 1.5%: 임의 단어로 랜덤 변경
  - 1.5%: 변경 X

2) **NSP**: 두 개의 문장이 주어지면, 이 문장이 이어지는 문장인지 아닌지를 맞추는 방식으로 훈련시킴.(50%는 이어지는 문장, 50%는 랜덤으로 선택된 안 이어지는 문장)


pretrain에서 사용하는 데이터셋은 위키피디아(25억 단어)와 Books Corpus(8억 단어)이다.

<br>

## 2. MLM(Masked LM)


```python
!pip install transformers
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting transformers
      Downloading transformers-4.26.1-py3-none-any.whl (6.3 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m6.3/6.3 MB[0m [31m19.9 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pyyaml>=5.1 in /usr/local/lib/python3.8/dist-packages (from transformers) (6.0)
    Requirement already satisfied: packaging>=20.0 in /usr/local/lib/python3.8/dist-packages (from transformers) (23.0)
    Requirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers) (3.9.0)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (2022.6.2)
    Collecting huggingface-hub<1.0,>=0.11.0
      Downloading huggingface_hub-0.12.0-py3-none-any.whl (190 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m190.3/190.3 KB[0m [31m11.3 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: tqdm>=4.27 in /usr/local/lib/python3.8/dist-packages (from transformers) (4.64.1)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers) (2.25.1)
    Requirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (1.21.6)
    Collecting tokenizers!=0.11.3,<0.14,>=0.11.1
      Downloading tokenizers-0.13.2-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (7.6 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m7.6/7.6 MB[0m [31m52.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: typing-extensions>=3.7.4.3 in /usr/local/lib/python3.8/dist-packages (from huggingface-hub<1.0,>=0.11.0->transformers) (4.4.0)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (1.24.3)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2022.12.7)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (4.0.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2.10)
    Installing collected packages: tokenizers, huggingface-hub, transformers
    Successfully installed huggingface-hub-0.12.0 tokenizers-0.13.2 transformers-4.26.1
    


```python
from transformers import BertTokenizer, BertForMaskedLM
import torch
```


```python
tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertForMaskedLM.from_pretrained('bert-base-uncased')

text = ("After Abraham Lincoln won the November 1860 presidential "
        "election on an anti-slavery platform, an initial seven "
        "slave states declared their secession from the country "
        "to form the Confederacy. War broke out in April 1861 "
        "when secessionist forces attacked Fort Sumter in South "
        "Carolina, just over a month after Lincoln's "
        "inauguration.")
```


    Downloading (…)solve/main/vocab.txt:   0%|          | 0.00/232k [00:00<?, ?B/s]



    Downloading (…)okenizer_config.json:   0%|          | 0.00/28.0 [00:00<?, ?B/s]



    Downloading (…)lve/main/config.json:   0%|          | 0.00/570 [00:00<?, ?B/s]



    Downloading (…)"pytorch_model.bin";:   0%|          | 0.00/440M [00:00<?, ?B/s]


    Some weights of the model checkpoint at bert-base-uncased were not used when initializing BertForMaskedLM: ['cls.seq_relationship.bias', 'cls.seq_relationship.weight']
    - This IS expected if you are initializing BertForMaskedLM from the checkpoint of a model trained on another task or with another architecture (e.g. initializing a BertForSequenceClassification model from a BertForPreTraining model).
    - This IS NOT expected if you are initializing BertForMaskedLM from the checkpoint of a model that you expect to be exactly identical (initializing a BertForSequenceClassification model from a BertForSequenceClassification model).
    


```python
inputs = tokenizer(text, return_tensors='pt') # return_tensors를 'pt'로 설정하면 pytorch tesnor 자료형으로 반환해준다.
```

토크나이저를 통과시키면 `input_ids`, `token_type_ids`, `attention_mask` 세 개의 임베딩을 반환해준다.

- `input_ids`: 각 단어를 정수 인코딩해준 것
- `token_type_ids`: 해당 단어가 첫번째 문장(0)인지 두번째 문장(1)인지를 나타내주는 것.
- `attention_mask`: attention layer가 무시해야 하는 패딩 토큰들은 0으로 지정, 아닌 것은 1로 지정



```python
inputs.keys()
```




    dict_keys(['input_ids', 'token_type_ids', 'attention_mask'])




```python
inputs['input_ids'] #101은 bos 토큰, 102는 eos 토큰이다.
```




    tensor([[  101,  2044,  8181,  5367,  2180,  1996,  2281,  7313,  4883,  2602,
              2006,  2019,  3424,  1011,  8864,  4132,  1010,  2019,  3988,  2698,
              6658,  2163,  4161,  2037, 22965,  2013,  1996,  2406,  2000,  2433,
              1996, 18179,  1012,  2162,  3631,  2041,  1999,  2258,  6863,  2043,
             22965,  2923,  2749,  4457,  3481,  7680,  3334,  1999,  2148,  3792,
              1010,  2074,  2058,  1037,  3204,  2044,  5367,  1005,  1055, 17331,
              1012,   102]])




```python
# 하나의 문장만 들어갔기 때문에 모든 단어가 0(첫번째 문장)으로 표시된다.
inputs['token_type_ids']
```




    tensor([[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]])




```python
# 패딩 토큰이 없기에, 모든 단어가 attention layer의 적용을 받도록 1로 표시된다.
inputs['attention_mask']
```




    tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
             1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
             1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]])




```python
inputs['labels'] = inputs.input_ids.detach().clone()
```

이제 특정 단어들에 Masking을 해줄 것이다. Mask는 전체 단어 중 15%에 한하여 진행한다. 그래서 단어들을 랜덤으로 0과 1사이 float으로 생성하고, 이 중 15%만 Masking을 해줄 것이다.


```python
# input_ids와 같은 차원의 랜덤 float array를 만든다.
rand = torch.rand(inputs.input_ids.shape)
# 이 때 0.15보다 작은 요소들을을 True로 만들어준다.
mask_arr = rand < 0.15
mask_arr
```




    tensor([[False, False, False,  True, False, False, False, False, False, False,
             False, False, False, False, False, False, False, False, False, False,
              True, False, False, False, False, False, False, False, False, False,
             False, False, False,  True,  True, False,  True, False, False, False,
              True, False, False,  True, False, False, False, False, False, False,
             False, False, False, False, False, False, False, False, False, False,
             False,  True]])




```python
# bos, eos도 False로 만들기 위해 아래처럼 조건을 더 건다.
mask_arr = (rand < 0.15) * (inputs.input_ids != 101) * (inputs.input_ids != 102)
mask_arr
```




    tensor([[False, False, False,  True, False, False, False, False, False, False,
             False, False, False, False, False, False, False, False, False, False,
              True, False, False, False, False, False, False, False, False, False,
             False, False, False,  True,  True, False,  True, False, False, False,
              True, False, False,  True, False, False, False, False, False, False,
             False, False, False, False, False, False, False, False, False, False,
             False, False]])




```python
# True인 값들의 index를 flatten하여 하나의 리스트에 담는다
selection = torch.flatten((mask_arr[0]).nonzero()).tolist()
selection
```




    [3, 20, 33, 34, 36, 40, 43]




```python
# Mask 토큰은 103으로 지정한다. 앞서 얻은 index들을 103으로 바꿔준다.
inputs.input_ids[0, selection] = 103
```


```python
inputs['input_ids'] # 103으로 변한 부분이 Mask 처리된 부분이다.
```




    tensor([[  101,  2044,  8181,   103,  2180,  1996,  2281,  7313,  4883,  2602,
              2006,  2019,  3424,  1011,  8864,  4132,  1010,  2019,  3988,  2698,
               103,  2163,  4161,  2037, 22965,  2013,  1996,  2406,  2000,  2433,
              1996, 18179,  1012,   103,   103,  2041,   103,  2258,  6863,  2043,
               103,  2923,  2749,   103,  3481,  7680,  3334,  1999,  2148,  3792,
              1010,  2074,  2058,  1037,  3204,  2044,  5367,  1005,  1055, 17331,
              1012,   102]])



이제 만들어진 inputs을 BERT 모델에 넣는다. loss 값을 반환해주는데, 이는 Mask된 부분의 단어에 한한 loss이다.


```python
outputs = model(**inputs)
```


```python
outputs.keys()
```




    odict_keys(['loss', 'logits'])




```python
outputs.loss
```




    tensor(0.8053, grad_fn=<NllLossBackward0>)




```python
outputs.logits
```




    tensor([[[ -7.1162,  -7.0597,  -7.0891,  ...,  -6.3137,  -6.1457,  -4.3020],
             [ -9.2592,  -9.0717,  -9.2359,  ...,  -8.8328,  -8.2930,  -7.9929],
             [ -8.1557,  -8.7103,  -8.2165,  ...,  -7.6762,  -6.8691,  -7.1768],
             ...,
             [ -1.4756,  -1.3714,  -1.3981,  ...,  -0.9416,  -0.5452,  -7.5062],
             [-13.9242, -13.8415, -13.8469,  ..., -10.8817, -11.1650,  -9.2146],
             [-11.8421, -12.2480, -11.7827,  ..., -11.7742,  -9.1722,  -9.1381]]],
           grad_fn=<ViewBackward0>)


<br>

## 3. NSP(Next Sentence Prediction)
방식은 MLM과 거의 유사하다. 다만 두 개의 문장을 입력으로 넣는다.


```python
from transformers import BertTokenizer, BertForNextSentencePrediction

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertForNextSentencePrediction.from_pretrained('bert-base-uncased')

text = ("After Abraham Lincoln won the November 1860 presidential election on an "
        "anti-slavery platform, an initial seven slave states declared their "
        "secession from the country to form the Confederacy.")
text2 = ("War broke out in April 1861 when secessionist forces attacked Fort "
         "Sumter in South Carolina, just over a month after Lincoln's "
         "inauguration.")
```

    Some weights of the model checkpoint at bert-base-uncased were not used when initializing BertForNextSentencePrediction: ['cls.predictions.transform.LayerNorm.bias', 'cls.predictions.bias', 'cls.predictions.transform.dense.bias', 'cls.predictions.transform.LayerNorm.weight', 'cls.predictions.transform.dense.weight', 'cls.predictions.decoder.weight']
    - This IS expected if you are initializing BertForNextSentencePrediction from the checkpoint of a model trained on another task or with another architecture (e.g. initializing a BertForSequenceClassification model from a BertForPreTraining model).
    - This IS NOT expected if you are initializing BertForNextSentencePrediction from the checkpoint of a model that you expect to be exactly identical (initializing a BertForSequenceClassification model from a BertForSequenceClassification model).
    


```python
inputs = tokenizer(text, text2, return_tensors='pt')
inputs.keys()
```




    dict_keys(['input_ids', 'token_type_ids', 'attention_mask'])




```python
inputs['token_type_ids'] # 문장을 두 개를 넣었으니 0(첫번째 문장)과 1(두번째 문장)로 구분이 된다.
```




    tensor([[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
             1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]])



label은 0(is next sentence)과 1(not next sentence)로 나뉘게 된다.


```python
labels = torch.LongTensor([0])
```


```python
outputs = model(**inputs, labels=labels)
outputs.keys()
```




    odict_keys(['loss', 'logits'])




```python
outputs.loss
```




    tensor(3.2186e-06, grad_fn=<NllLossBackward0>)



`logits`에서 반환되는 값을 보면 이어지는 문장이냐, 아니냐라는 이진 분류이기 때문에 값이 두 개만 있음을 확인할 수 있다.

앞서 input으로 넣은 두 문장은 이어지는 문장이었기 때문에 `argmax` 함수를 사용했을 때 0이 반환된다.


```python
outputs.logits
```




    tensor([[ 6.3968, -6.2638]], grad_fn=<AddmmBackward0>)




```python
torch.argmax(outputs.logits)
```




    tensor(0)



이번엔 영 연관이 없는 문장을 text2로 넣어보자.


```python
text = ("After Abraham Lincoln won the November 1860 presidential election on an "
        "anti-slavery platform, an initial seven slave states declared their "
        "secession from the country to form the Confederacy.")
text2 = ("Pele began playing for Santos at age 15 and the Brazil national team "
         "at 16. During his international career, he won three FIFA World Cups: 1958, 1962 and 1970, "
         "the only player to do so.")
```


```python
inputs = tokenizer(text, text2, return_tensors='pt')
```


```python
outputs = model(**inputs, labels=labels)
```


```python
outputs.loss
```




    tensor(12.9636, grad_fn=<NllLossBackward0>)




```python
outputs.logits
```




    tensor([[-5.0046,  7.9590]], grad_fn=<AddmmBackward0>)



그럼 1이 반환되며 두 문장이 이어지지 않는다는 것을 잘 분류해낸다.


```python
torch.argmax(outputs.logits)
```




    tensor(1)


