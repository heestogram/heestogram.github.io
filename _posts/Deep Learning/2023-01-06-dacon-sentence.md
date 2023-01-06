---
title: "[Pytorch] 문장 분류 연습(DACON 문장 유형 분류 대회 baseline 코드 연습)"
excerpt: "pytorch의 신경망과 TfidVecotorizer를 이용하여 문장 분류를 실습해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, pytorch, TfidfVectorizer]

published: true

categories:
  - DL

date: 2023-01-06 16:00:00
last_modified_at: 2023-01-06 16:00:00
---

<br>

자연어 처리(NLP)에 대해 공부를 할 필요를 느꼈다. 대표적으로 문장의 성격을 분류하는 분야가 가장 관심이 갔다. 적당한 챌린지가 DACON에 있길래, 이미 종료된 대회이지만 baseline 코드를 공부하며 pytorch로 NLP하는 것에 익숙해져보기로 했다.

[[DACON] 문장 유형 분류 AI 경진대회 안내 링크](https://dacon.io/competitions/official/236037/overview/description)

<br>

실습에 쓰인 데이터는 위 링크를 통해 다운로드 받을 수 있다. 나는 구글 코랩에서 작업했고 기본적으로 설치한 라이브러리는 다음과 같다.

## 기본 라이브러리 설치

```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
import random
import pandas as pd
import numpy as np
import os

from sklearn.model_selection import train_test_split
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.preprocessing import LabelEncoder
from sklearn.metrics import f1_score
from sklearn.metrics.pairwise import linear_kernel

import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F
from torch.utils.data import Dataset, DataLoader

from tqdm.auto import tqdm

import warnings
warnings.filterwarnings(action='ignore') 
```

pytorch를 할 때 GPU로 연산을 수행하려면 아래처럼 코드를 작성한다.

```python
device = torch.device('cuda') if torch.cuda.is_available() else torch.device('cpu')
```

아래 CFG에 기본 파라미터를 설정해주고 시드를 고정해준다.

```python
CFG = {
    'EPOCHS':10,
    'LEARNING_RATE':1e-4,
    'BATCH_SIZE':256,
    'SEED':41
}

def seed_everything(seed):
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = True

seed_everything(CFG['SEED']) # Seed 고정
```

<br>

## 데이터 불러오기

```python
df = pd.read_csv('/content/drive/MyDrive/sentence/train.csv')
test = pd.read_csv('/content/drive/MyDrive/sentence/test.csv')
```


```python
train, val, _, _ = train_test_split(df, df['label'], test_size=0.2, random_state=CFG['SEED'])
```

<br>

## TfidfVectorizer로 문장 전처리

`TfidfVectorizer()`는 TF-IDF(단어빈도*문서빈도역수)를 학습시키는 함수이다. 단어빈도란 특정 단어가 한 문서 내에서 출현한 빈도이고, 문서빈도는 특정 단어가 출현한 전체 문서의 개수이다. 왜 두 빈도를 모두 알아야 할까? 예컨대 정관사 a, the는 영어에서 굉장히 많이 쓰인다. 하지만 출현빈도에 비해 그렇게 중요치 않은 단어이다. 이처럼 단어빈도는 높아도 중요도가 낮은 단어들을 걸러주기 위해 이 단어가 출현한 문서의 빈도도 카운트해줄 필요가 있다는 얘기이다.


`min_df`는 최소 문서빈도를 설정해주는 파라미터이다. 아래처럼 이를 4로 설정하면, 4보다 낮은 문서빈도를 가진 단어는 `TfidfVectorizer()`의 학습 대상에서 제외된다.
`analyzer`는 학습단위를 설정해주는 파라미터이다. word로 설정하면 학습단위가 단어가 되고, char로 설정하면 학습단위가 철자가 된다. 
`ngram_range`는 단어의 묶음을 몇개까지 묶을 지 설정한다. 아래처럼 (1,2)라면 1개짜리 단어묶음과 2개짜리 단어묶음이 존재하게 되는데, 'I love', 'we like`처럼 두 개의 단어도 묶음이 될 수 있단 뜻이다.


```python
vectorizer = TfidfVectorizer(min_df = 4, analyzer = 'word', ngram_range=(1, 2))

df_vec = vectorizer.fit_transform(df["문장"])

train_vec = vectorizer.fit_transform(train["문장"])
val_vec = vectorizer.transform(val["문장"])
test_vec = vectorizer.transform(test["문장"])

print(train_vec.shape, val_vec.shape, test_vec.shape)
```

    (13232, 9351) (3309, 9351) (7090, 9351)
    

<br>

아래 코드의 출력값은 13,232개의 행이 9,351개의 단어로 이루어져 있고 0이 아닌 값은 총 108,346개가 존재한다는 의미이다.


```python
train_vec
```

    <13232x9351 sparse matrix of type '<class 'numpy.float64'>'
    	with 108346 stored elements in Compressed Sparse Row format>


<br>

첫번째 행(문장) 안에 9351개의 단어들이 0~1 사이 값으로 저장되어 있다. 거의 대두분의 값이 0일 것이므로 짧게 출력된 모습에선 0밖에 보이질 않는다.


```python
train_vec.toarray()[0]
```


    array([0., 0., 0., ..., 0., 0., 0.])

<br>

## 문장 간 유사도 확인

`linear_kernel()`로 문장간 유사도도 확인할 수 있다.


```python
# 문장 간 유사도를 계산하고 특정 문장에 대한 유사도 top n을 뽑아주는 함수

def similarity_rank(input_title, vec, data, n):
  cosine_sim = linear_kernel(vec, vec)
  indices = pd.Series(data.index, index=data['문장']).drop_duplicates()
  idx = indices[input_title]

  sim_scores = list(enumerate(cosine_sim[idx]))
  sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
  sim_scores_top_n = sim_scores[1:n+1]
  top_idx = [i[0] for i in sim_scores_top_n]

  return data['문장'].iloc[top_idx]
```


```python
input_title = "현재 의심환자의 검사 비용은 전액 국가 부담이다."

similarity_rank(input_title, df_vec, df, 10)
```




    7792     그러면서 정권에 충성하는 검사,반대로 정권에 저항하는 검사,범죄피해를 당한 검사,페...
    3314                          지역구는 현실이고 국가 전체는 이상이라는 고민이다.
    2023                            장인이 낸 운영 비용은 성주의 세금으로 쓰인다.
    12964    그러나 비트코인을 구입하는 데 사용한 비용은 약 1억300만(한화 약1328억원)으...
    10972    연간 관사 운영비는 2021년 11월 말 기준 1014만원으로 전액 도 예산으로 지...
    11015                                        피부 검사 방법도 같다.
    15489      김 부장은 한국외대를 졸업하고 2001년 서울지검 서부지청에서 검사 생활을 시작했다.
    5222     신종 코로나 관련 치료비는 국가에서 전액 지원함으로 치료비담보는 제외하였고, 기존 ...
    9225     또 캐롯은 보장 기간 종료 후 단기 질병안심보험 관련 정산이익(사차익) 발생시 전액...
    1720      두 사람의 낙찰 금액은 전액 손흥민 명의로 대한민국 축구종합센터 건립비용으로 기부된다.
    Name: 문장, dtype: object


<br>

## Label Encoding

우선 각 컬럼에 어떤 값들이 있는지 확인해본다.

```python
print(train["유형"].unique())
print(train["극성"].unique())
print(train["시제"].unique())
print(train["확실성"].unique())
```

    ['사실형' '추론형' '대화형' '예측형']
    ['긍정' '미정' '부정']
    ['과거' '현재' '미래']
    ['확실' '불확실']
    
위처럼 categorical한 변수들을 Label Encoding해주기로 한다.

```python
# categorical한 변수들을 labelencoding
type_le = LabelEncoder()
train["유형"] = type_le.fit_transform(train["유형"].values)
val["유형"] = type_le.transform(val["유형"].values)

polarity_le = LabelEncoder()
train["극성"] = polarity_le.fit_transform(train["극성"].values)
val["극성"] = polarity_le.transform(val["극성"].values)

tense_le = LabelEncoder()
train["시제"] = tense_le.fit_transform(train["시제"].values)
val["시제"] = tense_le.transform(val["시제"].values)

certainty_le = LabelEncoder()
train["확실성"] = certainty_le.fit_transform(train["확실성"].values)
val["확실성"] = certainty_le.transform(val["확실성"].values)
```


```python
train_type = train["유형"].values # sentence type
train_polarity = train["극성"].values # sentence polarity
train_tense = train["시제"].values # sentence tense
train_certainty = train["확실성"].values # sentence certainty

train_labels = {
    'type' : train_type,
    'polarity' : train_polarity,
    'tense' : train_tense,
    'certainty' : train_certainty
}

val_type = val["유형"].values # sentence type
val_polarity = val["극성"].values # sentence polarity
val_tense = val["시제"].values # sentence tense
val_certainty = val["확실성"].values # sentence certainty

val_labels = {
    'type' : val_type,
    'polarity' : val_polarity,
    'tense' : val_tense,
    'certainty' : val_certainty
}
```

<br>

## CustomDatset class 정의

pytorch를 다룰 때는 데이터셋을 불러올 때 가독성과 모듈성을 확보하기 위해 데이터셋 코드를 학습 코드로부터 분리한다. 그리고 이 데이터셋을 불러올 때 사용자 정의 클래스를 만드는 것이 일반적이다. 사용자 정의 클래스를 만들 때에는 반드시 `__init__`, `__getitem__`, `__len__` 메소드를 만들어주도록 한다.

`__init__`은 클래스를 생성하면 자동 실행되는 생성자로, 초기화를 담당한다.
`__getitem__`은 클래스의 인스턴스가 마치 리스트나 튜플처럼 슬라이싱이 가능하도록 구현하도록 하는 메소드이다.
`torch.FloatTensor()` 함수는 tensor 자료형으로 변환해주는 함수이다. 즉 아래 `__getitem__` 메소드는 데이터셋을 입력받았을 때 이를 torch 자료형으로 바꿔주고, 인덱싱이 가능하게끔 만들어준다.
`__len__`은 길이를 반환해주는 메소드이다.           


```python
class CustomDataset(Dataset):
    def __init__(self, st_vec, st_labels):
        self.st_vec = st_vec
        self.st_labels = st_labels

    def __getitem__(self, index):
        st_vector = torch.FloatTensor(self.st_vec[index].toarray()).squeeze(0)
        if self.st_labels is not None:
            st_type = self.st_labels['type'][index]
            st_polarity = self.st_labels['polarity'][index]
            st_tense = self.st_labels['tense'][index]
            st_certainty = self.st_labels['certainty'][index]
            return st_vector, st_type, st_polarity, st_tense, st_certainty
        else:
            return st_vector

    def __len__(self):
        return len(self.st_vec.toarray())
```

`DataLoader()`는 데이터셋을 미니배치 형태로 쪼개어주는 함수이다. 이로써 iterator 형식으로 데이터를 가공해준다.


```python
train_dataset = CustomDataset(train_vec, train_labels)
train_loader = DataLoader(train_dataset, batch_size = CFG['BATCH_SIZE'], shuffle=True, num_workers=0)

val_dataset = CustomDataset(val_vec, val_labels)
val_loader = DataLoader(val_dataset, batch_size = CFG['BATCH_SIZE'], shuffle=False, num_workers=0)
```

<br>

## pytorch로 신경망 모델 구축

이제 학습에 사용할 모델을 정의할 것이다. 전체적인 구조를 대략적으로 살펴보자. 우선 `__init__` 메소드에서 `feature_extract`라는 모델을 만들어서 기본적으로 변수를 받으면 그 변수를 심층망으로 분류해준다. 그리고나서 유형, 극성, 시제, 확실성의 네 가지 각각을 처리해주는 신경망을 구축한다. `feature_extract` 까지는 똑같은 코드로 작동하지만, 마지막에 out_features의 개수가 각기 다르므로 이렇게 네 개의 신경망을 달리 만들어준 것이다. 이후 `forward()` 메소드에서는 앞서 정의한 `feature_extract`와 네 가지 신경망을 각각 순전파로 연결해주고, 최종적으로 네 가지 신경망을 통과한 output 4개를 받게 된다.

이제 모델을 세세하게 살펴보자.
첫 줄을 보면 `nn.module`을 호출받고 있다는 점을 알 수 있다. 이 때, `__init__` 메소드에서 `super(Basemodel, self).__init_()`을 해줌으로써 호출 단계에서 부모 클래스(nn.Module)의 `__init__` 메소드를 호출해주고, 다양한 변수들을 상속받을 수 있는 것이다.
`input_dim`은 `vectorizer`로 생성한 단어들의 개수 9,351개이다.

`nn.linear`은 파이토치의 선형회귀 모델이다. `nn.BatchNorm1d`는 배치 정규화의 한 방식인데, 이렇게 각 레이어마다 정규화를 해주어야 학습이 안정적으로 이루어질 수 있다. 활성화 함수로는 `LeakyReLU()`가 쓰였다. `ReLU()`의 knockout 문제를 해결해주는 함수이다.

네 개의 각 신경망에서는 `nn.Dropuout(p=0.3)`을 설정해주며 30%의 확률로 뉴런을 제거하여 오버피팅을 억제하고 있다.


```python
class BaseModel(nn.Module):
    def __init__(self, input_dim=9351):
        super(BaseModel, self).__init__()
        self.feature_extract = nn.Sequential(
            nn.Linear(in_features=input_dim, out_features=1024),
            nn.BatchNorm1d(1024),
            nn.LeakyReLU(),
            nn.Linear(in_features=1024, out_features=1024),
            nn.BatchNorm1d(1024),
            nn.LeakyReLU(),
            nn.Linear(in_features=1024, out_features=512),
            nn.BatchNorm1d(512),
            nn.LeakyReLU(),
        )
        self.type_classifier = nn.Sequential(
            nn.Dropout(p=0.3),
            nn.Linear(in_features=512, out_features=4),
        )
        self.polarity_classifier = nn.Sequential(
            nn.Dropout(p=0.3),
            nn.Linear(in_features=512, out_features=3),
        )
        self.tense_classifier = nn.Sequential(
            nn.Dropout(p=0.3),
            nn.Linear(in_features=512, out_features=3),
        )
        self.certainty_classifier = nn.Sequential(
            nn.Dropout(p=0.3),
            nn.Linear(in_features=512, out_features=2),
        )
            
    def forward(self, x):
        x = self.feature_extract(x)
        # 문장 유형, 극성, 시제, 확실성을 각각 분류
        type_output = self.type_classifier(x)
        polarity_output = self.polarity_classifier(x)
        tense_output = self.tense_classifier(x)
        certainty_output = self.certainty_classifier(x)
        return type_output, polarity_output, tense_output, certainty_output
```

<br>

## 훈련 class 정의


이제 훈련을 시키는 train 함수를 정의하자. 코드가 길어지다보니 코드 내에서 주석으로 설명하는 편이 좋겠다.  



```python
def train(model, optimizer, train_loader, val_loader, scheduler, device):
    # .to(device)를 해줌으로써 device가 지정한 GPU에서 연산을 수행할 수 있다.
    model.to(device)
    
    criterion = {
        # nn.CrossEntropyLoss()는 다중 분류에서 자주쓰이는 손실함수이다.
        'type' : nn.CrossEntropyLoss().to(device),
        'polarity' : nn.CrossEntropyLoss().to(device),
        'tense' : nn.CrossEntropyLoss().to(device),
        'certainty' : nn.CrossEntropyLoss().to(device)
    }
    
    best_loss = 999999
    best_model = None
    
    for epoch in range(1, CFG['EPOCHS']+1):
        model.train()
        train_loss = []
        for sentence, type_label, polarity_label, tense_label, certainty_label in tqdm(iter(train_loader)):
            sentence = sentence.to(device)
            type_label = type_label.to(device)
            polarity_label = polarity_label.to(device)
            tense_label = tense_label.to(device)
            certainty_label = certainty_label.to(device)
            
            # 한 번의 학습이 완료되면 gradients값을 0으로 초기화 시켜줘야 한다. 
            optimizer.zero_grad()
            
            #앞서 정의한 BaseModel class를 실행시키는 것이다. 
            #결과값으론 신경망을 거친 type_output, polarity_output, tense_output, certainty_output가 나온다.
            type_logit, polarity_logit, tense_logit, certainty_logit = model(sentence)
            
            # criterion에서 어떤 손실함수를 쓸 지 정의해두었다.
            # 이 손실함수에 예측값(logit)과 실제 타겟값(label)을 인자로 넣어 loss를 계산한다.
            loss = 0.25 * criterion['type'](type_logit, type_label) + \
                    0.25 * criterion['polarity'](polarity_logit, polarity_label) + \
                    0.25 * criterion['tense'](tense_logit, tense_label) + \
                    0.25 * criterion['certainty'](certainty_logit, certainty_label)
            
            # 위에서 구한 loss값을 역전파하여 미분한다. 이로써 gradient값을 얻게 된다.
            loss.backward()

            # 위에서 얻은 gradient를 바탕으로 파라미터들을 최적화하는 작업이다.
            optimizer.step()
            
            # loss 값을 train_loss 리스트에 저장해두고 마지막에 print한다.
            train_loss.append(loss.item())
        
        # 한번의 epoch가 끝날 때마다 validation에 대한 점수도 출력을 한다.
        val_loss, val_type_f1, val_polarity_f1, val_tense_f1, val_certainty_f1 = validation(model, val_loader, criterion, device)
        print(f'Epoch : [{epoch}] Train Loss : [{np.mean(train_loss):.5f}] Val Loss : [{val_loss:.5f}] 유형 F1 : [{val_type_f1:.5f}] 극성 F1 : [{val_polarity_f1:.5f}] 시제 F1 : [{val_tense_f1:.5f}] 확실성 F1 : [{val_certainty_f1:.5f}]')
        
        # scheduler는 learning rate를 학습 과정에서 조정해주는 기능을 한다.
        if scheduler is not None:
            scheduler.step(val_loss)
            
        # 가장 낮은 val_loss를 기록한 model을 best_model로 저장한다.    
        if best_loss > val_loss:
            best_loss = val_loss
            best_model = model
            
    return best_model
```

<br>

## validation class 정의

pytorch의 validation 부분에서 꼭 등장하는 `model.eval()`에 대해 짚고 넘어가려고 한다. 간단히 말해 validation을 하는 과정에서 사용하면 안 되는 layer를 알아서 off시키는 함수이다. 우리는 검증(validation)이나 평가(evaluation)를 할 때 학습(train)하는 과정에서 쓰인 정규화나 각종 전처리 과정을 생략할 필요가 있다는 사실을 잘 안다. 이 같은 원리로, validation을 할 시에는 모든 노드를 정규화하지 않은 상태에서 사용할 것이라는 의미이다.

앞서 train 함수에서는 `model.train()`을 한 반면, validation 함수에서는 `model.eval()`을 한다. 그렇다면 validation에서 사용하면 안 되는 layer는 무엇일까? 바로 `Dropout`과 `BatchNorm1d`처럼 노드를 임의로 제거하거나 정규화시키는 layer들이다.

`with torch.no_grad()`는 pytorch의 autograd engine을 비활성화 시키는 기능이다. validation을 할 때엔 train을 할 때와 다르게 역전파를 필요로 하지 않기 때문에 gradient 값을 저장할 필요도 없다. 그래서 autograd engine을 비활성화시키는 것이다. 이렇게 하면 필요한 메모리가 줄어들고 연산이 빨라지는 효과를 볼 수 있다.


```python
# 앞서 정의한 train 함수에서 쓰일 validation 함수를 정의한다.

def validation(model, val_loader, criterion, device):
    model.eval()
    val_loss = []
    
    type_preds, polarity_preds, tense_preds, certainty_preds = [], [], [], []
    type_labels, polarity_labels, tense_labels, certainty_labels = [], [], [], []
    
    
    with torch.no_grad():
        for sentence, type_label, polarity_label, tense_label, certainty_label in tqdm(iter(val_loader)):
            sentence = sentence.to(device)
            type_label = type_label.to(device)
            polarity_label = polarity_label.to(device)
            tense_label = tense_label.to(device)
            certainty_label = certainty_label.to(device)
            
            type_logit, polarity_logit, tense_logit, certainty_logit = model(sentence)
            
            loss = 0.25 * criterion['type'](type_logit, type_label) + \
                    0.25 * criterion['polarity'](polarity_logit, polarity_label) + \
                    0.25 * criterion['tense'](tense_logit, tense_label) + \
                    0.25 * criterion['certainty'](certainty_logit, certainty_label)
            
            val_loss.append(loss.item())
            
            # 각 logit값들 중 최댓값인 것을 1로 만들어 label과 비교할 수 있도록 한다.
            type_preds += type_logit.argmax(1).detach().cpu().numpy().tolist()
            type_labels += type_label.detach().cpu().numpy().tolist()
            
            polarity_preds += polarity_logit.argmax(1).detach().cpu().numpy().tolist()
            polarity_labels += polarity_label.detach().cpu().numpy().tolist()
            
            tense_preds += tense_logit.argmax(1).detach().cpu().numpy().tolist()
            tense_labels += tense_label.detach().cpu().numpy().tolist()
            
            certainty_preds += certainty_logit.argmax(1).detach().cpu().numpy().tolist()
            certainty_labels += certainty_label.detach().cpu().numpy().tolist()
    
    # f1_score를 저장한다.
    type_f1 = f1_score(type_labels, type_preds, average='weighted')
    polarity_f1 = f1_score(polarity_labels, polarity_preds, average='weighted')
    tense_f1 = f1_score(tense_labels, tense_preds, average='weighted')
    certainty_f1 = f1_score(certainty_labels, certainty_preds, average='weighted')
    
    return np.mean(val_loss), type_f1, polarity_f1, tense_f1, certainty_f1
```

<br>

## 학습 RUN!

```python
model = BaseModel()
model.eval()

# pytorch에서 가장 자주 쓰이는 Adam optimizer이다.
optimizer = torch.optim.Adam(params = model.parameters(), lr = CFG["LEARNING_RATE"])

# ReduceLROnPlateau()는 모델의 개선이 없을 경우 learning rate를 조절해주는 콜백함수이다.
# mode는 목표값이 최소가 되어야하는지 최대가 되어야하는지 지정하는 파라미터이다.
# factor는 learning rate를 감소시키는 기준치로, 갱신된 learning rate는 기존 lr*factor이다.
# min_lr은 lr의 하한선이다.
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer, mode='min', factor=0.5, patience=2,threshold_mode='abs',min_lr=1e-8, verbose=True)

infer_model = train(model, optimizer, train_loader, val_loader, scheduler, device)
```


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [1] Train Loss : [0.85585] Val Loss : [0.63529] 유형 F1 : [0.73452] 극성 F1 : [0.93030] 시제 F1 : [0.49506] 확실성 F1 : [0.87274]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [2] Train Loss : [0.33244] Val Loss : [0.47759] 유형 F1 : [0.77251] 극성 F1 : [0.94140] 시제 F1 : [0.69921] 확실성 F1 : [0.87998]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [3] Train Loss : [0.16799] Val Loss : [0.42685] 유형 F1 : [0.78083] 극성 F1 : [0.94323] 시제 F1 : [0.71250] 확실성 F1 : [0.88623]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [4] Train Loss : [0.09804] Val Loss : [0.43498] 유형 F1 : [0.78630] 극성 F1 : [0.94776] 시제 F1 : [0.70317] 확실성 F1 : [0.89131]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [5] Train Loss : [0.06421] Val Loss : [0.43445] 유형 F1 : [0.79320] 극성 F1 : [0.94898] 시제 F1 : [0.71420] 확실성 F1 : [0.89299]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [6] Train Loss : [0.04663] Val Loss : [0.44376] 유형 F1 : [0.79042] 극성 F1 : [0.95203] 시제 F1 : [0.70613] 확실성 F1 : [0.89258]
    Epoch 00006: reducing learning rate of group 0 to 5.0000e-05.
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [7] Train Loss : [0.03716] Val Loss : [0.44708] 유형 F1 : [0.79110] 극성 F1 : [0.95342] 시제 F1 : [0.70788] 확실성 F1 : [0.89221]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [8] Train Loss : [0.03270] Val Loss : [0.45063] 유형 F1 : [0.79281] 극성 F1 : [0.95562] 시제 F1 : [0.70870] 확실성 F1 : [0.89241]
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [9] Train Loss : [0.02964] Val Loss : [0.45398] 유형 F1 : [0.79201] 극성 F1 : [0.95569] 시제 F1 : [0.70338] 확실성 F1 : [0.89361]
    Epoch 00009: reducing learning rate of group 0 to 2.5000e-05.
    


      0%|          | 0/52 [00:00<?, ?it/s]



      0%|          | 0/13 [00:00<?, ?it/s]


    Epoch : [10] Train Loss : [0.02657] Val Loss : [0.45785] 유형 F1 : [0.79354] 극성 F1 : [0.95598] 시제 F1 : [0.70488] 확실성 F1 : [0.89287]
    

<br>

## test set으로 inference하기

```python
test_dataset = CustomDataset(test_vec, None)
test_loader = DataLoader(test_dataset, batch_size=CFG['BATCH_SIZE'], shuffle=False, num_workers=0)
```


```python
def inference(model, test_loader, device):
    model.to(device)
    model.eval()
    
    type_preds, polarity_preds, tense_preds, certainty_preds = [], [], [], []
    
    with torch.no_grad():
        for sentence in tqdm(test_loader):
            sentence = sentence.to(device)
            
            type_logit, polarity_logit, tense_logit, certainty_logit = model(sentence)
            
            type_preds += type_logit.argmax(1).detach().cpu().numpy().tolist()
            polarity_preds += polarity_logit.argmax(1).detach().cpu().numpy().tolist()
            tense_preds += tense_logit.argmax(1).detach().cpu().numpy().tolist()
            certainty_preds += certainty_logit.argmax(1).detach().cpu().numpy().tolist()
            
    return type_preds, polarity_preds, tense_preds, certainty_preds
```


```python
type_preds, polarity_preds, tense_preds, certainty_preds = inference(model, test_loader, device)
```


      0%|          | 0/28 [00:00<?, ?it/s]

<br>

## 제출하기

```python
# 인코딩된 값들을 다시 원래대로 복구시킨다
type_preds = type_le.inverse_transform(type_preds)
polarity_preds = polarity_le.inverse_transform(polarity_preds)
tense_preds = tense_le.inverse_transform(tense_preds)
certainty_preds = certainty_le.inverse_transform(certainty_preds)
```


```python
predictions = []
for type_pred, polarity_pred, tense_pred, certainty_pred in zip(type_preds, polarity_preds, tense_preds, certainty_preds):
    predictions.append(type_pred+'-'+polarity_pred+'-'+tense_pred+'-'+certainty_pred)
```


```python
submit = pd.read_csv('/content/drive/MyDrive/sentence/sample_submission.csv')
submit['label'] = predictions
```


```python
submit.to_csv('./baseline_submit.csv', index=False)
```

<br>
