---
title: "[NLP] KoBART fine tuning 매뉴얼"
excerpt: "한국어 문장 요약에 특화된 KoBART를 원하는 데이터셋으로 fine tuning해보자."
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, KoBART, fine tuning]

published: true

categories:
  - DL

date: 2023-03-01 21:00:00
last_modified_at: 2023-03-01 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 contest '위키피디아&나무위키 요약 시스템 프로젝트'의 과정을 정리한 것입니다.
</div>

<br>

본 코드는 구글 코랩 환경에서 실행되었습니다.

<br>

## 실행하기 전에 런타임 유형을 꼭!! GPU로 바꿔놔야 함.

## fine tuning 데이터셋 만들기

우선 적당한 데이터셋을 찾아야 한다. AI-Hub에서 제공하는 논문요약 자료를 훈련 데이터셋으로 쓰기로 했다. 적당한 구글 드라이브 디렉터리에 파일을 다운받아놓는다.

source from: https://aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100&aihubDataSe=realm&dataSetSn=582



```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
import json
import pandas as pd
import matplotlib.pyplot as plt
import os
from tqdm import tqdm
```

저장해놓은 파일을 json으로 불러온다.


```python
with open("/content/drive/MyDrive/kubig/kucon/논문요약_0225_5_1.json", "r") as st_json:
    data = json.load(st_json)
```


```python
print("데이터 개수:", data['totalcount']) # 데이터 크기는 32,000개다.
```

    데이터 개수: 32000
    

json 형식을 데이터프레임으로 바꿔주는 함수이다.


```python
def json_to_df(data):
  id = []
  category = []
  text = []
  summary = []
  for i in range(data['totalcount']):
    id.append(data['data'][i]['doc_id'])
    category.append(data['data'][i]['ipc'])
    text.append(data['data'][i]['summary_entire'][0]['orginal_text'])
    summary.append(data['data'][i]['summary_entire'][0]['summary_text'])
  df = pd.DataFrame({'id':id,
                     'category':category,
                      'text':text,
                     'summary':summary})
  return df
```

32,000개 데이터를 모두 fine tuning 시키면 1 epoch당 1시간 넘게 소요가 돼서 일단 여기서는 100개의 데이터만 예시로 사용해볼 것이다.


```python
df = json_to_df(data)
df_ = df[:100] # fine-tuning이 너무 길어지지 않게 일부만 사용하기로
```


```python
# train, test 분리

length_data = len(df_)     # data 행 개수
split_ratio = 0.7           # 0.7 / 0.3 으로 분리
length_train = round(length_data * split_ratio)  
length_validation = length_data - length_train
print("Data length :", length_data)
print("Train data length :", length_train)
print("Validation data lenth :", length_validation)
```

    Data length : 100
    Train data length : 70
    Validation data lenth : 30
    

컬럼명을 아래처럼 정의해야 fine tuning에서 에러가 안 난다.


```python
df_ = df_.iloc[:,2:]
df_.columns = ['news','summary']

train = df_[:length_train]
test = df_[length_train:]
```

fine tuning의 데이터는 tsv 파일 형태로 넣어줘야 하기 때문에 아래코드를 실행한다.

저장이 된 파일들은 일단 보관해둔다.


```python
train.to_csv('train.tsv', sep='\t', encoding='utf-8')
```


```python
test.to_csv('test.tsv', sep='\t', encoding='utf-8')
```

<br>

## fine tuning에 사용할 레퍼지터리 clone



```python
!pip install git+https://github.com/SKT-AI/KoBART
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting git+https://github.com/SKT-AI/KoBART
      Cloning https://github.com/SKT-AI/KoBART to /tmp/pip-req-build-6t3xrzpf
      Running command git clone --filter=blob:none --quiet https://github.com/SKT-AI/KoBART /tmp/pip-req-build-6t3xrzpf
      Resolved https://github.com/SKT-AI/KoBART to commit 30c5eb7b593828d6ec2d767eeedb2f2ed02c5c2a
      Preparing metadata (setup.py) ... [?25l[?25hdone
    Collecting boto3
      Downloading boto3-1.26.74-py3-none-any.whl (132 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m132.7/132.7 KB[0m [31m11.3 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pandas in /usr/local/lib/python3.8/dist-packages (from kobart==0.5.1) (1.3.5)
    Collecting pytorch-lightning==1.2.1
      Downloading pytorch_lightning-1.2.1-py3-none-any.whl (814 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m814.2/814.2 KB[0m [31m45.2 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting torch==1.7.1
      Downloading torch-1.7.1-cp38-cp38-manylinux1_x86_64.whl (776.8 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m776.8/776.8 MB[0m [31m1.9 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting transformers==4.3.3
      Downloading transformers-4.3.3-py3-none-any.whl (1.9 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m1.9/1.9 MB[0m [31m74.3 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: tensorboard>=2.2.0 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.2.1->kobart==0.5.1) (2.11.2)
    Requirement already satisfied: PyYAML!=5.4.*,>=5.1 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.2.1->kobart==0.5.1) (6.0)
    Collecting future>=0.17.1
      Downloading future-0.18.3.tar.gz (840 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m840.9/840.9 KB[0m [31m54.0 MB/s[0m eta [36m0:00:00[0m
    [?25h  Preparing metadata (setup.py) ... [?25l[?25hdone
    Requirement already satisfied: numpy>=1.16.6 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.2.1->kobart==0.5.1) (1.21.6)
    Requirement already satisfied: tqdm>=4.41.0 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.2.1->kobart==0.5.1) (4.64.1)
    Requirement already satisfied: fsspec[http]>=0.8.1 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.2.1->kobart==0.5.1) (2023.1.0)
    Requirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from torch==1.7.1->kobart==0.5.1) (4.5.0)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers==4.3.3->kobart==0.5.1) (2022.6.2)
    Collecting tokenizers<0.11,>=0.10.1
      Downloading tokenizers-0.10.3-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (3.3 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m3.3/3.3 MB[0m [31m84.6 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting sacremoses
      Downloading sacremoses-0.0.53.tar.gz (880 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m880.6/880.6 KB[0m [31m55.6 MB/s[0m eta [36m0:00:00[0m
    [?25h  Preparing metadata (setup.py) ... [?25l[?25hdone
    Requirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers==4.3.3->kobart==0.5.1) (3.9.0)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers==4.3.3->kobart==0.5.1) (2.25.1)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from transformers==4.3.3->kobart==0.5.1) (23.0)
    Collecting s3transfer<0.7.0,>=0.6.0
      Downloading s3transfer-0.6.0-py3-none-any.whl (79 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m79.6/79.6 KB[0m [31m8.5 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting botocore<1.30.0,>=1.29.74
      Downloading botocore-1.29.74-py3-none-any.whl (10.4 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m10.4/10.4 MB[0m [31m84.6 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting jmespath<2.0.0,>=0.7.1
      Downloading jmespath-1.0.1-py3-none-any.whl (20 kB)
    Requirement already satisfied: pytz>=2017.3 in /usr/local/lib/python3.8/dist-packages (from pandas->kobart==0.5.1) (2022.7.1)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.8/dist-packages (from pandas->kobart==0.5.1) (2.8.2)
    Collecting urllib3<1.27,>=1.25.4
      Downloading urllib3-1.26.14-py2.py3-none-any.whl (140 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m140.6/140.6 KB[0m [31m16.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: aiohttp!=4.0.0a0,!=4.0.0a1 in /usr/local/lib/python3.8/dist-packages (from fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (3.8.4)
    Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.8/dist-packages (from python-dateutil>=2.7.3->pandas->kobart==0.5.1) (1.15.0)
    Requirement already satisfied: grpcio>=1.24.3 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (1.51.1)
    Requirement already satisfied: werkzeug>=1.0.1 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (1.0.1)
    Requirement already satisfied: markdown>=2.6.8 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (3.4.1)
    Requirement already satisfied: tensorboard-plugin-wit>=1.6.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (1.8.1)
    Requirement already satisfied: google-auth<3,>=1.6.3 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (2.16.0)
    Requirement already satisfied: wheel>=0.26 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (0.38.4)
    Requirement already satisfied: google-auth-oauthlib<0.5,>=0.4.1 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (0.4.6)
    Requirement already satisfied: tensorboard-data-server<0.7.0,>=0.6.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (0.6.1)
    Requirement already satisfied: absl-py>=0.4 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (1.4.0)
    Requirement already satisfied: setuptools>=41.0.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (57.4.0)
    Requirement already satisfied: protobuf<4,>=3.9.2 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (3.19.6)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.3.3->kobart==0.5.1) (4.0.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.3.3->kobart==0.5.1) (2.10)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.3.3->kobart==0.5.1) (2022.12.7)
    Requirement already satisfied: click in /usr/local/lib/python3.8/dist-packages (from sacremoses->transformers==4.3.3->kobart==0.5.1) (7.1.2)
    Requirement already satisfied: joblib in /usr/local/lib/python3.8/dist-packages (from sacremoses->transformers==4.3.3->kobart==0.5.1) (1.2.0)
    Requirement already satisfied: async-timeout<5.0,>=4.0.0a3 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (4.0.2)
    Requirement already satisfied: frozenlist>=1.1.1 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (1.3.3)
    Requirement already satisfied: charset-normalizer<4.0,>=2.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (3.0.1)
    Requirement already satisfied: aiosignal>=1.1.2 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (1.3.1)
    Requirement already satisfied: yarl<2.0,>=1.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (1.8.2)
    Requirement already satisfied: multidict<7.0,>=4.5 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (6.0.4)
    Requirement already satisfied: attrs>=17.3.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]>=0.8.1->pytorch-lightning==1.2.1->kobart==0.5.1) (22.2.0)
    Requirement already satisfied: pyasn1-modules>=0.2.1 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (0.2.8)
    Requirement already satisfied: rsa<5,>=3.1.4 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (4.9)
    Requirement already satisfied: cachetools<6.0,>=2.0.0 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (5.3.0)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in /usr/local/lib/python3.8/dist-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (1.3.1)
    Requirement already satisfied: importlib-metadata>=4.4 in /usr/local/lib/python3.8/dist-packages (from markdown>=2.6.8->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (6.0.0)
    Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.8/dist-packages (from importlib-metadata>=4.4->markdown>=2.6.8->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (3.13.0)
    Requirement already satisfied: pyasn1<0.5.0,>=0.4.6 in /usr/local/lib/python3.8/dist-packages (from pyasn1-modules>=0.2.1->google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (0.4.8)
    Requirement already satisfied: oauthlib>=3.0.0 in /usr/local/lib/python3.8/dist-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard>=2.2.0->pytorch-lightning==1.2.1->kobart==0.5.1) (3.2.2)
    Building wheels for collected packages: kobart, future, sacremoses
      Building wheel for kobart (setup.py) ... [?25l[?25hdone
      Created wheel for kobart: filename=kobart-0.5.1-py3-none-any.whl size=9584 sha256=10597b0507a95bdb04db23e4f20a4ea9069d6c2f2e72bb5877351c81f5bb54ff
      Stored in directory: /tmp/pip-ephem-wheel-cache-6xl_7pld/wheels/08/af/7d/32d2d2d08c8009730f02412f3d62a364a2c1bf477803f9183f
      Building wheel for future (setup.py) ... [?25l[?25hdone
      Created wheel for future: filename=future-0.18.3-py3-none-any.whl size=492036 sha256=a2a31e16f3a310b8fd5454d8b250dad6cab03db103ac7cc8ab3cfb2c17375b75
      Stored in directory: /root/.cache/pip/wheels/a0/0b/ee/e6994fadb42c1354dcccb139b0bf2795271bddfe6253ccdf11
      Building wheel for sacremoses (setup.py) ... [?25l[?25hdone
      Created wheel for sacremoses: filename=sacremoses-0.0.53-py3-none-any.whl size=895260 sha256=870e4ff7d461db778f6ecfd95cbf5c44f281a4b362cf186c7741fb7603e8a270
      Stored in directory: /root/.cache/pip/wheels/82/ab/9b/c15899bf659ba74f623ac776e861cf2eb8608c1825ddec66a4
    Successfully built kobart future sacremoses
    Installing collected packages: tokenizers, urllib3, torch, sacremoses, jmespath, future, botocore, transformers, s3transfer, boto3, pytorch-lightning, kobart
      Attempting uninstall: urllib3
        Found existing installation: urllib3 1.24.3
        Uninstalling urllib3-1.24.3:
          Successfully uninstalled urllib3-1.24.3
      Attempting uninstall: torch
        Found existing installation: torch 1.13.1+cu116
        Uninstalling torch-1.13.1+cu116:
          Successfully uninstalled torch-1.13.1+cu116
      Attempting uninstall: future
        Found existing installation: future 0.16.0
        Uninstalling future-0.16.0:
          Successfully uninstalled future-0.16.0
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    torchvision 0.14.1+cu116 requires torch==1.13.1, but you have torch 1.7.1 which is incompatible.
    torchtext 0.14.1 requires torch==1.13.1, but you have torch 1.7.1 which is incompatible.
    torchaudio 0.13.1+cu116 requires torch==1.13.1, but you have torch 1.7.1 which is incompatible.[0m[31m
    [0mSuccessfully installed boto3-1.26.74 botocore-1.29.74 future-0.18.3 jmespath-1.0.1 kobart-0.5.1 pytorch-lightning-1.2.1 s3transfer-0.6.0 sacremoses-0.0.53 tokenizers-0.10.3 torch-1.7.1 transformers-4.3.3 urllib3-1.26.14
    


```python
!pip install transformers
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Requirement already satisfied: transformers in /usr/local/lib/python3.8/dist-packages (4.3.3)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from transformers) (23.0)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (2022.6.2)
    Requirement already satisfied: sacremoses in /usr/local/lib/python3.8/dist-packages (from transformers) (0.0.53)
    Requirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (1.21.6)
    Requirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers) (3.9.0)
    Requirement already satisfied: tokenizers<0.11,>=0.10.1 in /usr/local/lib/python3.8/dist-packages (from transformers) (0.10.3)
    Requirement already satisfied: tqdm>=4.27 in /usr/local/lib/python3.8/dist-packages (from transformers) (4.64.1)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers) (2.25.1)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2022.12.7)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (4.0.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2.10)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (1.26.14)
    Requirement already satisfied: click in /usr/local/lib/python3.8/dist-packages (from sacremoses->transformers) (7.1.2)
    Requirement already satisfied: joblib in /usr/local/lib/python3.8/dist-packages (from sacremoses->transformers) (1.2.0)
    Requirement already satisfied: six in /usr/local/lib/python3.8/dist-packages (from sacremoses->transformers) (1.15.0)
    

내 로컬의 원하는 폴더에서 git bash를 켜고

`$ git clone https://github.com/seujung/KoBART-summarization.git` 명령어를 입력해준다.

폴더가 잘 clone이 되었다면 이 폴더를 구글 드라이브 원하는 위치에 넣어준다.

바로 드라이브에 clone하는 방법도 있는지는 잘 모르겠다.

그리고선 cd 명령어로 폴더를 넣어준 디렉터리로 이동한다


```python
%cd /content/drive/MyDrive/kubig/kucon/KoBART-summarization
```

    /content/drive/MyDrive/kubig/kucon/KoBART-summarization
    

<br>

## fine tuning 코드 실행 전 환경 설정

훈련에 필요한 환경을 설치해준다. `requirements.txt`파일은 clone해온 폴더 내에 들어 있다.


```python
!pip install -r requirements.txt
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Requirement already satisfied: pandas in /usr/local/lib/python3.8/dist-packages (from -r requirements.txt (line 1)) (1.3.5)
    Collecting torch==1.10.0
      Downloading torch-1.10.0-cp38-cp38-manylinux1_x86_64.whl (881.9 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m881.9/881.9 MB[0m [31m1.7 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting transformers==4.8.2
      Downloading transformers-4.8.2-py3-none-any.whl (2.5 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m2.5/2.5 MB[0m [31m66.7 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting pytorch-lightning==1.3.8
      Downloading pytorch_lightning-1.3.8-py3-none-any.whl (813 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m813.4/813.4 KB[0m [31m55.3 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting streamlit==1.1.0
      Downloading streamlit-1.1.0-py2.py3-none-any.whl (8.3 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m8.3/8.3 MB[0m [31m88.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from torch==1.10.0->-r requirements.txt (line 2)) (4.5.0)
    Requirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (3.9.0)
    Requirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (1.21.6)
    Requirement already satisfied: tqdm>=4.27 in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (4.64.1)
    Requirement already satisfied: tokenizers<0.11,>=0.10.1 in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (0.10.3)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (2.25.1)
    Requirement already satisfied: pyyaml in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (6.0)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (23.0)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (2022.6.2)
    Collecting huggingface-hub==0.0.12
      Downloading huggingface_hub-0.0.12-py3-none-any.whl (37 kB)
    Requirement already satisfied: sacremoses in /usr/local/lib/python3.8/dist-packages (from transformers==4.8.2->-r requirements.txt (line 3)) (0.0.53)
    Requirement already satisfied: future>=0.17.1 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (0.18.3)
    Collecting pyDeprecate==0.3.0
      Downloading pyDeprecate-0.3.0-py3-none-any.whl (10 kB)
    Requirement already satisfied: tensorboard!=2.5.0,>=2.2.0 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (2.11.2)
    Requirement already satisfied: pillow!=8.3.0 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (7.1.2)
    Collecting pyyaml
      Downloading PyYAML-5.4.1-cp38-cp38-manylinux1_x86_64.whl (662 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m662.4/662.4 KB[0m [31m49.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: fsspec[http]!=2021.06.0,>=2021.05.0 in /usr/local/lib/python3.8/dist-packages (from pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (2023.1.0)
    Collecting torchmetrics>=0.2.0
      Downloading torchmetrics-0.11.1-py3-none-any.whl (517 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m517.2/517.2 KB[0m [31m44.2 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting pydeck>=0.1.dev5
      Downloading pydeck-0.8.0-py2.py3-none-any.whl (4.7 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m4.7/4.7 MB[0m [31m76.0 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: astor in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (0.8.1)
    Collecting base58
      Downloading base58-2.1.1-py3-none-any.whl (5.6 kB)
    Requirement already satisfied: tornado>=5.0 in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (6.2)
    Requirement already satisfied: python-dateutil in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (2.8.2)
    Requirement already satisfied: altair>=3.2.0 in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (4.2.2)
    Requirement already satisfied: protobuf!=3.11,>=3.6.0 in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (3.19.6)
    Requirement already satisfied: click<8.0,>=7.0 in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (7.1.2)
    Requirement already satisfied: tzlocal in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (1.5.1)
    Collecting blinker
      Downloading blinker-1.5-py2.py3-none-any.whl (12 kB)
    Requirement already satisfied: attrs in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (22.2.0)
    Collecting gitpython!=3.1.19
      Downloading GitPython-3.1.31-py3-none-any.whl (184 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m184.3/184.3 KB[0m [31m18.0 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pyarrow in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (9.0.0)
    Requirement already satisfied: toml in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (0.10.2)
    Requirement already satisfied: cachetools>=4.0 in /usr/local/lib/python3.8/dist-packages (from streamlit==1.1.0->-r requirements.txt (line 5)) (5.3.0)
    Collecting validators
      Downloading validators-0.20.0.tar.gz (30 kB)
      Preparing metadata (setup.py) ... [?25l[?25hdone
    Collecting watchdog
      Downloading watchdog-2.2.1-py3-none-manylinux2014_x86_64.whl (78 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m79.0/79.0 KB[0m [31m9.2 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pytz>=2017.3 in /usr/local/lib/python3.8/dist-packages (from pandas->-r requirements.txt (line 1)) (2022.7.1)
    Requirement already satisfied: toolz in /usr/local/lib/python3.8/dist-packages (from altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (0.12.0)
    Requirement already satisfied: jsonschema>=3.0 in /usr/local/lib/python3.8/dist-packages (from altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (4.3.3)
    Requirement already satisfied: jinja2 in /usr/local/lib/python3.8/dist-packages (from altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (2.11.3)
    Requirement already satisfied: entrypoints in /usr/local/lib/python3.8/dist-packages (from altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (0.4)
    Requirement already satisfied: aiohttp!=4.0.0a0,!=4.0.0a1 in /usr/local/lib/python3.8/dist-packages (from fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (3.8.4)
    Collecting gitdb<5,>=4.0.1
      Downloading gitdb-4.0.10-py3-none-any.whl (62 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m62.7/62.7 KB[0m [31m5.6 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: six>=1.5 in /usr/local/lib/python3.8/dist-packages (from python-dateutil->streamlit==1.1.0->-r requirements.txt (line 5)) (1.15.0)
    Requirement already satisfied: setuptools>=41.0.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (57.4.0)
    Requirement already satisfied: google-auth-oauthlib<0.5,>=0.4.1 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (0.4.6)
    Requirement already satisfied: google-auth<3,>=1.6.3 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (2.16.0)
    Requirement already satisfied: werkzeug>=1.0.1 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.0.1)
    Requirement already satisfied: grpcio>=1.24.3 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.51.1)
    Requirement already satisfied: tensorboard-plugin-wit>=1.6.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.8.1)
    Requirement already satisfied: markdown>=2.6.8 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (3.4.1)
    Requirement already satisfied: wheel>=0.26 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (0.38.4)
    Requirement already satisfied: tensorboard-data-server<0.7.0,>=0.6.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (0.6.1)
    Requirement already satisfied: absl-py>=0.4 in /usr/local/lib/python3.8/dist-packages (from tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.4.0)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.8.2->-r requirements.txt (line 3)) (1.26.14)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.8.2->-r requirements.txt (line 3)) (2022.12.7)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.8.2->-r requirements.txt (line 3)) (4.0.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers==4.8.2->-r requirements.txt (line 3)) (2.10)
    Requirement already satisfied: joblib in /usr/local/lib/python3.8/dist-packages (from sacremoses->transformers==4.8.2->-r requirements.txt (line 3)) (1.2.0)
    Requirement already satisfied: decorator>=3.4.0 in /usr/local/lib/python3.8/dist-packages (from validators->streamlit==1.1.0->-r requirements.txt (line 5)) (4.4.2)
    Requirement already satisfied: yarl<2.0,>=1.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.8.2)
    Requirement already satisfied: frozenlist>=1.1.1 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.3.3)
    Requirement already satisfied: charset-normalizer<4.0,>=2.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (3.0.1)
    Requirement already satisfied: multidict<7.0,>=4.5 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (6.0.4)
    Requirement already satisfied: aiosignal>=1.1.2 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.3.1)
    Requirement already satisfied: async-timeout<5.0,>=4.0.0a3 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (4.0.2)
    Collecting smmap<6,>=3.0.1
      Downloading smmap-5.0.0-py3-none-any.whl (24 kB)
    Requirement already satisfied: rsa<5,>=3.1.4 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (4.9)
    Requirement already satisfied: pyasn1-modules>=0.2.1 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (0.2.8)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in /usr/local/lib/python3.8/dist-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (1.3.1)
    Requirement already satisfied: MarkupSafe>=0.23 in /usr/local/lib/python3.8/dist-packages (from jinja2->altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (2.0.1)
    Requirement already satisfied: pyrsistent!=0.17.0,!=0.17.1,!=0.17.2,>=0.14.0 in /usr/local/lib/python3.8/dist-packages (from jsonschema>=3.0->altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (0.19.3)
    Requirement already satisfied: importlib-resources>=1.4.0 in /usr/local/lib/python3.8/dist-packages (from jsonschema>=3.0->altair>=3.2.0->streamlit==1.1.0->-r requirements.txt (line 5)) (5.10.2)
    Requirement already satisfied: importlib-metadata>=4.4 in /usr/local/lib/python3.8/dist-packages (from markdown>=2.6.8->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (6.0.0)
    Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.8/dist-packages (from importlib-metadata>=4.4->markdown>=2.6.8->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (3.13.0)
    Requirement already satisfied: pyasn1<0.5.0,>=0.4.6 in /usr/local/lib/python3.8/dist-packages (from pyasn1-modules>=0.2.1->google-auth<3,>=1.6.3->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (0.4.8)
    Requirement already satisfied: oauthlib>=3.0.0 in /usr/local/lib/python3.8/dist-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard!=2.5.0,>=2.2.0->pytorch-lightning==1.3.8->-r requirements.txt (line 4)) (3.2.2)
    Building wheels for collected packages: validators
      Building wheel for validators (setup.py) ... [?25l[?25hdone
      Created wheel for validators: filename=validators-0.20.0-py3-none-any.whl size=19581 sha256=18b3fad3eb4c3f99c90f19cc4d3a2814224e223a66e0fcf2734d475af94e46eb
      Stored in directory: /root/.cache/pip/wheels/19/09/72/3eb74d236bb48bd0f3c6c3c83e4e0c5bbfcbcad7c6c3539db8
    Successfully built validators
    Installing collected packages: watchdog, validators, torch, smmap, pyyaml, pyDeprecate, blinker, base58, torchmetrics, pydeck, huggingface-hub, gitdb, transformers, gitpython, streamlit, pytorch-lightning
      Attempting uninstall: torch
        Found existing installation: torch 1.7.1
        Uninstalling torch-1.7.1:
          Successfully uninstalled torch-1.7.1
      Attempting uninstall: pyyaml
        Found existing installation: PyYAML 6.0
        Uninstalling PyYAML-6.0:
          Successfully uninstalled PyYAML-6.0
      Attempting uninstall: transformers
        Found existing installation: transformers 4.3.3
        Uninstalling transformers-4.3.3:
          Successfully uninstalled transformers-4.3.3
      Attempting uninstall: pytorch-lightning
        Found existing installation: pytorch-lightning 1.2.1
        Uninstalling pytorch-lightning-1.2.1:
          Successfully uninstalled pytorch-lightning-1.2.1
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    torchvision 0.14.1+cu116 requires torch==1.13.1, but you have torch 1.10.0 which is incompatible.
    torchtext 0.14.1 requires torch==1.13.1, but you have torch 1.10.0 which is incompatible.
    torchaudio 0.13.1+cu116 requires torch==1.13.1, but you have torch 1.10.0 which is incompatible.
    kobart 0.5.1 requires pytorch-lightning==1.2.1, but you have pytorch-lightning 1.3.8 which is incompatible.
    kobart 0.5.1 requires torch==1.7.1, but you have torch 1.10.0 which is incompatible.
    kobart 0.5.1 requires transformers==4.3.3, but you have transformers 4.8.2 which is incompatible.[0m[31m
    [0mSuccessfully installed base58-2.1.1 blinker-1.5 gitdb-4.0.10 gitpython-3.1.31 huggingface-hub-0.0.12 pyDeprecate-0.3.0 pydeck-0.8.0 pytorch-lightning-1.3.8 pyyaml-5.4.1 smmap-5.0.0 streamlit-1.1.0 torch-1.10.0 torchmetrics-0.11.1 transformers-4.8.2 validators-0.20.0 watchdog-2.2.1
    

그리고, 앞서 만든 tsv 파일을 `/KoBART-summarization/data` 위치에 저장해준다.

사실 fine tuning은 코드 한 줄이면 바로 실행할 수 있지만, 그 전에 코드 몇 개를 더 실행해야 오류가 안 나는 것 같다. 물론 작업환경마다 발생하는 오류의 종류도 다를 수 있기에 아래 방식이 잘 안 먹힐 수도 있다.


```python
!pip install torch==1.7.1+cu101 torchvision==0.8.2+cu101 -f https://download.pytorch.org/whl/torch_stable.html
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Looking in links: https://download.pytorch.org/whl/torch_stable.html
    Collecting torch==1.7.1+cu101
      Downloading https://download.pytorch.org/whl/cu101/torch-1.7.1%2Bcu101-cp38-cp38-linux_x86_64.whl (735.4 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m735.4/735.4 MB[0m [31m2.0 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting torchvision==0.8.2+cu101
      Downloading https://download.pytorch.org/whl/cu101/torchvision-0.8.2%2Bcu101-cp38-cp38-linux_x86_64.whl (12.8 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m12.8/12.8 MB[0m [31m22.2 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from torch==1.7.1+cu101) (4.5.0)
    Requirement already satisfied: numpy in /usr/local/lib/python3.8/dist-packages (from torch==1.7.1+cu101) (1.21.6)
    Requirement already satisfied: pillow>=4.1.1 in /usr/local/lib/python3.8/dist-packages (from torchvision==0.8.2+cu101) (7.1.2)
    Installing collected packages: torch, torchvision
      Attempting uninstall: torch
        Found existing installation: torch 1.10.0
        Uninstalling torch-1.10.0:
          Successfully uninstalled torch-1.10.0
      Attempting uninstall: torchvision
        Found existing installation: torchvision 0.14.1+cu116
        Uninstalling torchvision-0.14.1+cu116:
          Successfully uninstalled torchvision-0.14.1+cu116
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    torchtext 0.14.1 requires torch==1.13.1, but you have torch 1.7.1+cu101 which is incompatible.
    torchmetrics 0.11.1 requires torch>=1.8.1, but you have torch 1.7.1+cu101 which is incompatible.
    torchaudio 0.13.1+cu116 requires torch==1.13.1, but you have torch 1.7.1+cu101 which is incompatible.
    kobart 0.5.1 requires pytorch-lightning==1.2.1, but you have pytorch-lightning 1.3.8 which is incompatible.
    kobart 0.5.1 requires transformers==4.3.3, but you have transformers 4.8.2 which is incompatible.[0m[31m
    [0mSuccessfully installed torch-1.7.1+cu101 torchvision-0.8.2+cu101
    


```python
!nvidia-smi
```

    NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
    
    

아래는 `ImportError: cannot import name 'get_num_classes' from 'torchmetrics.utilities.data' (/usr/local/lib/python3.8/dist-packages/torchmetrics/utilities/data.py)`
오류를 디버깅하는 코드이다.


```python
!pip install torchmetrics==0.6.0
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting torchmetrics==0.6.0
      Downloading torchmetrics-0.6.0-py3-none-any.whl (329 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m329.4/329.4 KB[0m [31m21.8 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: numpy>=1.17.2 in /usr/local/lib/python3.8/dist-packages (from torchmetrics==0.6.0) (1.21.6)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from torchmetrics==0.6.0) (23.0)
    Requirement already satisfied: torch>=1.3.1 in /usr/local/lib/python3.8/dist-packages (from torchmetrics==0.6.0) (1.7.1+cu101)
    Requirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from torch>=1.3.1->torchmetrics==0.6.0) (4.5.0)
    Installing collected packages: torchmetrics
      Attempting uninstall: torchmetrics
        Found existing installation: torchmetrics 0.11.1
        Uninstalling torchmetrics-0.11.1:
          Successfully uninstalled torchmetrics-0.11.1
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    kobart 0.5.1 requires pytorch-lightning==1.2.1, but you have pytorch-lightning 1.3.8 which is incompatible.
    kobart 0.5.1 requires transformers==4.3.3, but you have transformers 4.8.2 which is incompatible.[0m[31m
    [0mSuccessfully installed torchmetrics-0.6.0
    


아래는 `OSError: /usr/local/lib/python3.8/dist-packages/torchtext/lib/libtorchtext.so: undefined symbol:` 오류를 디버깅 하는 코드이다.

torch와 torchtext를 맞춰주는 과정이다.


```python
!pip install torchtext==0.8.0
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting torchtext==0.8.0
      Downloading torchtext-0.8.0-cp38-cp38-manylinux1_x86_64.whl (7.0 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m7.0/7.0 MB[0m [31m38.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: tqdm in /usr/local/lib/python3.8/dist-packages (from torchtext==0.8.0) (4.64.1)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from torchtext==0.8.0) (2.25.1)
    Requirement already satisfied: torch in /usr/local/lib/python3.8/dist-packages (from torchtext==0.8.0) (1.7.1+cu101)
    Requirement already satisfied: numpy in /usr/local/lib/python3.8/dist-packages (from torchtext==0.8.0) (1.21.6)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->torchtext==0.8.0) (2022.12.7)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->torchtext==0.8.0) (4.0.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->torchtext==0.8.0) (2.10)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->torchtext==0.8.0) (1.26.14)
    Requirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from torch->torchtext==0.8.0) (4.5.0)
    Installing collected packages: torchtext
      Attempting uninstall: torchtext
        Found existing installation: torchtext 0.14.1
        Uninstalling torchtext-0.14.1:
          Successfully uninstalled torchtext-0.14.1
    Successfully installed torchtext-0.8.0
    

아래는 `AttributeError: 'Trainer' object has no attribute '_data_connector'` 오류를 디버깅하는 코드이다.

오류 발생 원인은 pytorch_lightning의 버전이 호환되지 않아서이다.


```python
!pip install pytorch_lightning==1.5.2
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting pytorch_lightning==1.5.2
      Downloading pytorch_lightning-1.5.2-py3-none-any.whl (1.0 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m1.0/1.0 MB[0m [31m31.4 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: tensorboard>=2.2.0 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (2.11.2)
    Requirement already satisfied: torch>=1.6 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (1.7.1+cu101)
    Requirement already satisfied: packaging>=17.0 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (23.0)
    Requirement already satisfied: torchmetrics>=0.4.1 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (0.6.0)
    Requirement already satisfied: tqdm>=4.41.0 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (4.64.1)
    Requirement already satisfied: future>=0.17.1 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (0.18.3)
    Requirement already satisfied: fsspec[http]!=2021.06.0,>=2021.05.0 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (2023.1.0)
    Requirement already satisfied: typing-extensions in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (4.5.0)
    Requirement already satisfied: numpy>=1.17.2 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (1.21.6)
    Collecting pyDeprecate==0.3.1
      Downloading pyDeprecate-0.3.1-py3-none-any.whl (10 kB)
    Requirement already satisfied: PyYAML>=5.1 in /usr/local/lib/python3.8/dist-packages (from pytorch_lightning==1.5.2) (5.4.1)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (2.25.1)
    Requirement already satisfied: aiohttp!=4.0.0a0,!=4.0.0a1 in /usr/local/lib/python3.8/dist-packages (from fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (3.8.4)
    Requirement already satisfied: google-auth<3,>=1.6.3 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (2.16.0)
    Requirement already satisfied: absl-py>=0.4 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (1.4.0)
    Requirement already satisfied: markdown>=2.6.8 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (3.4.1)
    Requirement already satisfied: werkzeug>=1.0.1 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (1.0.1)
    Requirement already satisfied: tensorboard-plugin-wit>=1.6.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (1.8.1)
    Requirement already satisfied: setuptools>=41.0.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (57.4.0)
    Requirement already satisfied: wheel>=0.26 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (0.38.4)
    Requirement already satisfied: protobuf<4,>=3.9.2 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (3.19.6)
    Requirement already satisfied: tensorboard-data-server<0.7.0,>=0.6.0 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (0.6.1)
    Requirement already satisfied: google-auth-oauthlib<0.5,>=0.4.1 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (0.4.6)
    Requirement already satisfied: grpcio>=1.24.3 in /usr/local/lib/python3.8/dist-packages (from tensorboard>=2.2.0->pytorch_lightning==1.5.2) (1.51.1)
    Requirement already satisfied: async-timeout<5.0,>=4.0.0a3 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (4.0.2)
    Requirement already satisfied: charset-normalizer<4.0,>=2.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (3.0.1)
    Requirement already satisfied: multidict<7.0,>=4.5 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (6.0.4)
    Requirement already satisfied: frozenlist>=1.1.1 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (1.3.3)
    Requirement already satisfied: aiosignal>=1.1.2 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (1.3.1)
    Requirement already satisfied: attrs>=17.3.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (22.2.0)
    Requirement already satisfied: yarl<2.0,>=1.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp!=4.0.0a0,!=4.0.0a1->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (1.8.2)
    Requirement already satisfied: cachetools<6.0,>=2.0.0 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (5.3.0)
    Requirement already satisfied: pyasn1-modules>=0.2.1 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (0.2.8)
    Requirement already satisfied: six>=1.9.0 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (1.15.0)
    Requirement already satisfied: rsa<5,>=3.1.4 in /usr/local/lib/python3.8/dist-packages (from google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (4.9)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in /usr/local/lib/python3.8/dist-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (1.3.1)
    Requirement already satisfied: importlib-metadata>=4.4 in /usr/local/lib/python3.8/dist-packages (from markdown>=2.6.8->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (6.0.0)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (1.26.14)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (2.10)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (4.0.0)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->fsspec[http]!=2021.06.0,>=2021.05.0->pytorch_lightning==1.5.2) (2022.12.7)
    Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.8/dist-packages (from importlib-metadata>=4.4->markdown>=2.6.8->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (3.13.0)
    Requirement already satisfied: pyasn1<0.5.0,>=0.4.6 in /usr/local/lib/python3.8/dist-packages (from pyasn1-modules>=0.2.1->google-auth<3,>=1.6.3->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (0.4.8)
    Requirement already satisfied: oauthlib>=3.0.0 in /usr/local/lib/python3.8/dist-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard>=2.2.0->pytorch_lightning==1.5.2) (3.2.2)
    Installing collected packages: pyDeprecate, pytorch_lightning
      Attempting uninstall: pyDeprecate
        Found existing installation: pyDeprecate 0.3.0
        Uninstalling pyDeprecate-0.3.0:
          Successfully uninstalled pyDeprecate-0.3.0
      Attempting uninstall: pytorch_lightning
        Found existing installation: pytorch-lightning 1.3.8
        Uninstalling pytorch-lightning-1.3.8:
          Successfully uninstalled pytorch-lightning-1.3.8
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    kobart 0.5.1 requires pytorch-lightning==1.2.1, but you have pytorch-lightning 1.5.2 which is incompatible.
    kobart 0.5.1 requires transformers==4.3.3, but you have transformers 4.8.2 which is incompatible.[0m[31m
    [0mSuccessfully installed pyDeprecate-0.3.1 pytorch_lightning-1.5.2
    

<br>

## fine tuning 시작!


```python
# GPU 사용시 쓰는 코드이다.
# 이 코드가 fine-tuning하는 코드. epoch이나 batch_size를 조정해줘도 된다.
!python train.py  --gradient_clip_val 1.0 --max_epochs 5 --default_root_dir logs --gpus 1 --batch_size 8 --num_workers 4
```

위 코드가 정상적으로 종료됐다면 logs 파일에 각 epoch별 ckpt(체크포인트) 파일이 저장됐을 것이다.

아래 코드는 저장된 모델의 체크포인트를 불러와 모델을 bin 파일로 설정해주는 작업이다.

hparams의 경우에는 ./logs/tb_logs/default/version_0/hparams.yaml 파일을 활용하고,

model_binary 의 경우에는 ./logs/model_chp 안에 있는 .ckpt 파일을 활용하면 되는데, loss 낮은 것을 임의대로 선택하면 될 것 같다.


```python
!python get_model_binary.py --hparams ./logs/tb_logs/default/version_0/hparams.yaml --model_binary ./logs/model_chp/epoch=01-val_loss=0.717.ckpt
```

    2023-02-20 06:48:09.374980: I tensorflow/core/platform/cpu_feature_guard.cc:193] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
    To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    2023-02-20 06:48:13.297680: W tensorflow/compiler/xla/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libnvinfer.so.7'; dlerror: libnvinfer.so.7: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /usr/local/nvidia/lib:/usr/local/nvidia/lib64
    2023-02-20 06:48:13.298006: W tensorflow/compiler/xla/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libnvinfer_plugin.so.7'; dlerror: libnvinfer_plugin.so.7: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /usr/local/nvidia/lib:/usr/local/nvidia/lib64
    2023-02-20 06:48:13.298033: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Cannot dlopen some TensorRT libraries. If you would like to use Nvidia GPU with TensorRT, please make sure the missing libraries mentioned above are installed properly.
    get_model_binary.py:13: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
      hparams = yaml.load(f)
    Downloading: 100% 1.36k/1.36k [00:00<00:00, 1.15MB/s]
    Downloading: 100% 496M/496M [00:08<00:00, 60.8MB/s]
    Downloading: 100% 4.00/4.00 [00:00<00:00, 2.31kB/s]
    Downloading: 100% 111/111 [00:00<00:00, 63.8kB/s]
    Downloading: 100% 682k/682k [00:00<00:00, 21.5MB/s]
    

위 코드가 정상적으로 작동했다면 kobart_summary 폴더 내에 config.json 파일과 pytorch_model.bin 파일이 생성된다.


`streamlit`은 python으로 데모 웹을 만들어주는 모듈인데, 원래 아래 코드를 실행하면 URL이 뜨고 이를 누르면 데모 페이지가 뜨는 게 정상이다.

근데 URL은 만들어지는데 데모 페이지가 열리지가 않는 상황!


```python
# 모델을 Demo Page로 열어주는 코드
!streamlit run infer.py
```

    2023-02-20 06:51:07.384 INFO    numexpr.utils: NumExpr defaulting to 2 threads.
    [0m
    [34m[1m  You can now view your Streamlit app in your browser.[0m
    [0m
    [34m  Network URL: [0m[1mhttp://172.28.0.12:8501[0m
    [34m  External URL: [0m[1mhttp://35.245.58.120:8501[0m
    [0m
    /content/drive/MyDrive/kubig/kucon/KoBART-summarization/.cache/kobart_base_tokenizer_cased_cf74400bce.zip[██████████████████████████████████████████████████]
    [34m  Stopping...[0m
    [34m  Stopping...[0m
    
