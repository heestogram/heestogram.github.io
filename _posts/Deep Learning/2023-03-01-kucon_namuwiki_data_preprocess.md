---
title: "[NLP] huggingface 나무위키 데이터셋 전처리"
excerpt: "huggingface에서 제공하는 나무위키 데이터셋을 전처리해보자"
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

본 코드는 허깅페이스에서 나무위키 데이터셋을 불러오고, 이를 전처리하여 요약 시스템을 구현하기 위한 시행착오입니다.

<br>

데이터 출처
https://huggingface.co/datasets/heegyu/namuwiki


## 1. 데이터 불러오기


```python
!pip install datasets
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting datasets
      Downloading datasets-2.9.0-py3-none-any.whl (462 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m462.8/462.8 KB[0m [31m8.4 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pyyaml>=5.1 in /usr/local/lib/python3.8/dist-packages (from datasets) (6.0)
    Requirement already satisfied: requests>=2.19.0 in /usr/local/lib/python3.8/dist-packages (from datasets) (2.25.1)
    Requirement already satisfied: aiohttp in /usr/local/lib/python3.8/dist-packages (from datasets) (3.8.4)
    Requirement already satisfied: fsspec[http]>=2021.11.1 in /usr/local/lib/python3.8/dist-packages (from datasets) (2023.1.0)
    Requirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from datasets) (1.21.6)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from datasets) (23.0)
    Collecting responses<0.19
      Downloading responses-0.18.0-py3-none-any.whl (38 kB)
    Requirement already satisfied: pandas in /usr/local/lib/python3.8/dist-packages (from datasets) (1.3.5)
    Requirement already satisfied: tqdm>=4.62.1 in /usr/local/lib/python3.8/dist-packages (from datasets) (4.64.1)
    Collecting xxhash
      Downloading xxhash-3.2.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (213 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m213.0/213.0 KB[0m [31m2.6 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: dill<0.3.7 in /usr/local/lib/python3.8/dist-packages (from datasets) (0.3.6)
    Requirement already satisfied: pyarrow>=6.0.0 in /usr/local/lib/python3.8/dist-packages (from datasets) (9.0.0)
    Collecting huggingface-hub<1.0.0,>=0.2.0
      Downloading huggingface_hub-0.12.1-py3-none-any.whl (190 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m190.3/190.3 KB[0m [31m11.4 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting multiprocess
      Downloading multiprocess-0.70.14-py38-none-any.whl (132 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m132.0/132.0 KB[0m [31m9.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: async-timeout<5.0,>=4.0.0a3 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (4.0.2)
    Requirement already satisfied: multidict<7.0,>=4.5 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (6.0.4)
    Requirement already satisfied: frozenlist>=1.1.1 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (1.3.3)
    Requirement already satisfied: yarl<2.0,>=1.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (1.8.2)
    Requirement already satisfied: charset-normalizer<4.0,>=2.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (3.0.1)
    Requirement already satisfied: aiosignal>=1.1.2 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (1.3.1)
    Requirement already satisfied: attrs>=17.3.0 in /usr/local/lib/python3.8/dist-packages (from aiohttp->datasets) (22.2.0)
    Requirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from huggingface-hub<1.0.0,>=0.2.0->datasets) (3.9.0)
    Requirement already satisfied: typing-extensions>=3.7.4.3 in /usr/local/lib/python3.8/dist-packages (from huggingface-hub<1.0.0,>=0.2.0->datasets) (4.5.0)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests>=2.19.0->datasets) (1.24.3)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests>=2.19.0->datasets) (2.10)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests>=2.19.0->datasets) (4.0.0)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests>=2.19.0->datasets) (2022.12.7)
    Collecting urllib3<1.27,>=1.21.1
      Downloading urllib3-1.26.14-py2.py3-none-any.whl (140 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m140.6/140.6 KB[0m [31m10.4 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.8/dist-packages (from pandas->datasets) (2.8.2)
    Requirement already satisfied: pytz>=2017.3 in /usr/local/lib/python3.8/dist-packages (from pandas->datasets) (2022.7.1)
    Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.8/dist-packages (from python-dateutil>=2.7.3->pandas->datasets) (1.15.0)
    Installing collected packages: xxhash, urllib3, multiprocess, responses, huggingface-hub, datasets
      Attempting uninstall: urllib3
        Found existing installation: urllib3 1.24.3
        Uninstalling urllib3-1.24.3:
          Successfully uninstalled urllib3-1.24.3
    Successfully installed datasets-2.9.0 huggingface-hub-0.12.1 multiprocess-0.70.14 responses-0.18.0 urllib3-1.26.14 xxhash-3.2.0
    


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
import pandas as pd
import matplotlib.pyplot as plt
import math
import re
```


```python
from datasets import load_dataset

data = load_dataset("heegyu/namuwiki")
```


    Downloading readme:   0%|          | 0.00/6.19k [00:00<?, ?B/s]


    WARNING:datasets.builder:Using custom data configuration heegyu--namuwiki-ad416814e2c61654
    

    Downloading and preparing dataset parquet/heegyu--namuwiki to /root/.cache/huggingface/datasets/heegyu___parquet/heegyu--namuwiki-ad416814e2c61654/0.0.0/2a3b91fbd88a2c90d1dbbb32b460cf621d31bd5b05b934492fdef7d8d6f236ec...
    


    Downloading data files:   0%|          | 0/1 [00:00<?, ?it/s]



    Downloading data:   0%|          | 0.00/3.03G [00:00<?, ?B/s]


    WARNING:datasets.download.download_manager:Computing checksums of downloaded files. They can be used for integrity verification. You can disable this by passing ignore_verifications=True to load_dataset
    


    Computing checksums: 100%|##########| 1/1 [00:11<00:00, 11.33s/it]



    Extracting data files:   0%|          | 0/1 [00:00<?, ?it/s]



    Generating train split: 0 examples [00:00, ? examples/s]


    Dataset parquet downloaded and prepared to /root/.cache/huggingface/datasets/heegyu___parquet/heegyu--namuwiki-ad416814e2c61654/0.0.0/2a3b91fbd88a2c90d1dbbb32b460cf621d31bd5b05b934492fdef7d8d6f236ec. Subsequent calls will reuse this data.
    


      0%|          | 0/1 [00:00<?, ?it/s]



```python
# 데이터 구성 확인
# title과 text만 필요할 것 같다.
data
```




    DatasetDict({
        train: Dataset({
            features: ['title', 'text', 'contributors', 'namespace'],
            num_rows: 867024
        })
    })




```python
dic = {'title':data['train']['title'],
       'text':data['train']['text']}

df = pd.DataFrame(dic)
```


```python
df.isnull().sum()
```




    title    0
    text     0
    dtype: int64




```python
df.head()
```





  <div id="df-eec37611-67ed-4be1-bf27-a68278d54f31">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>!</td>
      <td>#redirect 느낌표\n</td>
    </tr>
    <tr>
      <th>1</th>
      <td>!!아앗!!</td>
      <td>\n[목차]\n\n'''{{{+1 ！！ああっと！！}}}'''\n\n== 개요 ==\...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>“……”</td>
      <td>||&lt;-2&gt;&lt;tablebordercolor=#878787&gt;&lt;tablealign=ri...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>#</td>
      <td>[[분류:특수 문자]]\n[include(틀:다른 뜻1, other1=음악에서 사용...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>#FairyJoke</td>
      <td>[include(틀:링크시 주의, 링크=[[\\#FairyJoke]] 또는 [[#F...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-eec37611-67ed-4be1-bf27-a68278d54f31')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-eec37611-67ed-4be1-bf27-a68278d54f31 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-eec37611-67ed-4be1-bf27-a68278d54f31');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>


<br>

## 2. 불필요한 데이터 삭제

text 길이가 너무 짧은 문서는 요약이 무의미하기도 하고, 애당초 큰 영양가 없는 문서일 확률이 크며, 불필요한 연산을 야기하므로 없애기로 한다.

없앨 길이의 기준을 몇으로 잡을 지가 관건이다.

러프하게 기준치를 잡아놓고 요약 모델에 넣었을때 그럴싸하게 요약이 되는지 확인해보기.

예를 들어, 길이가 200인 text를 KoBART에 넣어보고 괜찮게 요약이 된다 싶으면, 190도 넣어보고 210도 넣어보면서 적당한 기준값을 잡아보기.


```python
def text_length(df):
  length = 0
  for i in df['text']:
    length+=len(i)
  print("text 길이 평균:", length/len(df))

text_length(df)
```

    text 길이 평균: 4949.858721327207
    


```python
def text_length_hist(df):
  x = df['text'].str.len()
  plt.xlim([0,10000])
  plt.hist(x, bins=500)
  plt.xlabel('text length')
  plt.title('text length distribution')
  plt.show()

text_length_hist(df)

# 1000보다 낮은 쪽에 굉장히 몰려있다.
```


    
<img src="https://user-images.githubusercontent.com/115082062/229093379-a660af5a-7434-4d15-8867-cc8317279f5a.png">

<br>
    


KoBART 모델을 불러와서 기준치 잡아보기


```python
!pip install transformers
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting transformers
      Downloading transformers-4.26.1-py3-none-any.whl (6.3 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m6.3/6.3 MB[0m [31m55.8 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers) (3.9.0)
    Requirement already satisfied: huggingface-hub<1.0,>=0.11.0 in /usr/local/lib/python3.8/dist-packages (from transformers) (0.12.1)
    Requirement already satisfied: packaging>=20.0 in /usr/local/lib/python3.8/dist-packages (from transformers) (23.0)
    Requirement already satisfied: tqdm>=4.27 in /usr/local/lib/python3.8/dist-packages (from transformers) (4.64.1)
    Requirement already satisfied: pyyaml>=5.1 in /usr/local/lib/python3.8/dist-packages (from transformers) (6.0)
    Requirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (1.21.6)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers) (2.25.1)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (2022.6.2)
    Collecting tokenizers!=0.11.3,<0.14,>=0.11.1
      Downloading tokenizers-0.13.2-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (7.6 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m7.6/7.6 MB[0m [31m38.1 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: typing-extensions>=3.7.4.3 in /usr/local/lib/python3.8/dist-packages (from huggingface-hub<1.0,>=0.11.0->transformers) (4.4.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2.10)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2022.12.7)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (1.26.14)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (4.0.0)
    Installing collected packages: tokenizers, transformers
    Successfully installed tokenizers-0.13.2 transformers-4.26.1
    


```python
import torch
from transformers import PreTrainedTokenizerFast
from transformers import BartForConditionalGeneration

tokenizer = PreTrainedTokenizerFast.from_pretrained('digit82/kobart-summarization')
model = BartForConditionalGeneration.from_pretrained('digit82/kobart-summarization')
```


    Downloading (…)okenizer_config.json:   0%|          | 0.00/295 [00:00<?, ?B/s]



    Downloading (…)/main/tokenizer.json:   0%|          | 0.00/682k [00:00<?, ?B/s]



    Downloading (…)cial_tokens_map.json:   0%|          | 0.00/109 [00:00<?, ?B/s]



    Downloading (…)lve/main/config.json:   0%|          | 0.00/1.20k [00:00<?, ?B/s]


    You passed along `num_labels=3` with an incompatible id to label map: {'0': 'NEGATIVE', '1': 'POSITIVE'}. The number of labels wil be overwritten to 2.
    


    Downloading (…)"pytorch_model.bin";:   0%|          | 0.00/496M [00:00<?, ?B/s]



```python
text = "세종은 32년의 재위 치세 동안 수많은 치적을 남겨 조선을 대표하는 최고의 성군으로 칭송받는다. \
세종이 창제한 한글은 현대의 대한민국/북한의 공용문자로 지정되어 통용되고 있으며 \
세종 시대에 확립된 북방의 국경은 그대로 한반도 이북 지역의 국경으로 자리잡아 오늘날까지 이어진다. \
그만큼 세종의 치세는 현대 한국인의 문화와 생활에도 큰 영향을 끼쳤으며, \
이러한 업적으로 인해 세종은 이순신과 함께 한국인에게 가장 존경받는 인물이 되었다."

print("원문 text 길이:",len(text))

raw_input_ids = tokenizer.encode(text) # 토큰화, 정수 인코딩
input_ids = [tokenizer.bos_token_id] + raw_input_ids + [tokenizer.eos_token_id] # bos, eos 토큰 추가
summary_ids = model.generate(torch.tensor([input_ids]),  num_beams=4,  max_length=512,  eos_token_id=1)
result = tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
print("요약문:",result)
print("요약문 text 길이:",len(result))
```

    원문 text 길이: 242
    요약문: 32년의 재위 치세 동안 수많은 치적을 남겨 조선을 대표하는 최고의 성군으로 칭송받는 세종이 창제한 한글은 현대의 대한민국/북한의 공용문자로 지정되어 통용되고 있으며 세종 시대에 확립된 북방의 국경은 그대로 한반도 이북 지역의 국경으로 자리잡아 오늘날까지 이어진다.
    요약문 text 길이: 148
    


```python
text="미드는 1863년 미국 메사추세츠에서 목사 가문의 아들로 태어났다. 부모님 모두 학문적 성향이 있었으며 [2] \
부친이 오벌린 대학에서 출강하기 위해 1870년 오하이오로 이주한다. 그러나 겨우 10년 뒤 그가 겨우 고등학생일 무렵 \
부친이 사망하여 가족들은 집을 팔고 셋집으로 이사할 수밖에 없었다. 1883년 오벌린 대학을 졸업했으나 곧바로 학자의 길을 걷지 않는다. \
오히려 졸업 이후 6개월간 고등학교 교사 생활을 했으며 약 4년간은 철도 측량사나 가정교사로 일했다.졸업 후 4년이나 지난 \
1887년 하버드 대학원에 등록하여 공부를 시작한다. 이 때의 전공은 철학. 대학원 생활 중 지적 관심을 철학에서 사회심리학 \
쪽으로 옮기게 되며, 대학원 두 번째 해에 독일 라이프치히로 건너가 분트의 실험실연구를 접하며 그 다음 해엔 베를린에서 게오르그 \
짐멜의 강연을 듣는다. 베를린에선 심리학 뿐만 아니라 각종 독일 철학을 접하며 그의 사회심리학 사상에 철학을 접목시킨다."

print("원문 text 길이:",len(text))

raw_input_ids = tokenizer.encode(text) # 토큰화, 정수 인코딩
input_ids = [tokenizer.bos_token_id] + raw_input_ids + [tokenizer.eos_token_id] # bos, eos 토큰 추가
summary_ids = model.generate(torch.tensor([input_ids]),  num_beams=4,  max_length=512,  eos_token_id=1)
result = tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
print("요약문:",result)
print("요약문 text 길이:",len(result))
```

    원문 text 길이: 484
    요약문: 1863년 미국 메사추세츠에서 목사 가문의 아들로 태어난 미드는 1863년 미국 메사추세츠에서 목사 가문의 아들로 태어났으며 1870년 오벌린 대학에서 출강하기 위해 1870년 오하이오로 이주했다.
    요약문 text 길이: 110
    

242를 넣으니 148로 줄여준다. 문장 2개 중 하나만 없어지고 거의 원문과 그대로인 수준이다.

400정도가 마지노선인 것 같다. 그보다 짧으면 요약의 효과가 미미한 듯하다.

거기다 보통 text를 보면 불필요한 특수기호들이 많으므로 이런 것을 감안해서 500자를 기준으로 잡아도 될 듯.

그러나 어디까지나 주관적이고 rough한 기준이므로, 기준을 새로 잡을 필요 있으니 나중에 논의해봄직 하다!


```python
def ratio_cal(length):
  x = df[df.apply(lambda x: len(x['text'])<length, axis=1)]
  ratio = x.shape[0]/df.shape[0]
  print("text길이가 {0} 미만인 것은 전체의 {1}".format(length,round(ratio,4)))
```


```python
ratio_cal(400)
ratio_cal(250)
ratio_cal(150)
```

    text길이가 400 미만인 것은 전체의 0.3613
    text길이가 250 미만인 것은 전체의 0.3483
    text길이가 150 미만인 것은 전체의 0.3418
    


```python
def slicing_df(df,length):
  x = df[df.apply(lambda x: len(x['text'])>=length, axis=1)]
  return x

# text 길이가 500 이하인 것은 없애기
sliced_df = slicing_df(df,500)
```


```python
long_df = slicing_df(df,150000)
```


```python
long_df.shape[0]
```




    516




```python
long_df.head()
```





  <div id="df-c1c25223-2777-41e9-8c87-91cc7ae25803">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8719</th>
      <td>Don't Starve/가공 아이템</td>
      <td>[include(틀:Don't Starve/관련 문서)]\n\n[목차]\n\n== ...</td>
    </tr>
    <tr>
      <th>8721</th>
      <td>Don't Starve/몬스터</td>
      <td>[include(틀:상위 문서, top1=Don't Starve)]\n[Includ...</td>
    </tr>
    <tr>
      <th>8722</th>
      <td>Don't Starve/식품</td>
      <td>[include(틀:Don't Starve/관련 문서)]\n\n[목차]\n\n== ...</td>
    </tr>
    <tr>
      <th>9608</th>
      <td>FC Schalke 04 Esports</td>
      <td>[include(틀:LEC 참가팀)]\n----\n||&lt;-2&gt;&lt;tableborder...</td>
    </tr>
    <tr>
      <th>10515</th>
      <td>Fate Another/캐릭터</td>
      <td>[include(틀:회원수정)]\n[include(틀:상위 문서, top1=Fate...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-c1c25223-2777-41e9-8c87-91cc7ae25803')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-c1c25223-2777-41e9-8c87-91cc7ae25803 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-c1c25223-2777-41e9-8c87-91cc7ae25803');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
sliced_df.head()
```





  <div id="df-cd3c2774-a5e2-48e3-b05b-26243897fb4e">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>!!아앗!!</td>
      <td>\n[목차]\n\n'''{{{+1 ！！ああっと！！}}}'''\n\n== 개요 ==\...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>“……”</td>
      <td>||&lt;-2&gt;&lt;tablebordercolor=#878787&gt;&lt;tablealign=ri...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>#</td>
      <td>[[분류:특수 문자]]\n[include(틀:다른 뜻1, other1=음악에서 사용...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>#FairyJoke</td>
      <td>[include(틀:링크시 주의, 링크=[[\\#FairyJoke]] 또는 [[#F...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>#Fairy_dancing_in_lake</td>
      <td>[include(틀:링크시 주의, 링크=[[\\#Fairy_dancing_in_la...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-cd3c2774-a5e2-48e3-b05b-26243897fb4e')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-cd3c2774-a5e2-48e3-b05b-26243897fb4e button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-cd3c2774-a5e2-48e3-b05b-26243897fb4e');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
sliced_df.shape[0]
```




    488115



원하는 제목을 입력해서 잘 뽑히는지 확인


```python
# 검색어가 포함된 제목 모두 출력
def search_title(df, target):
  searched = df[df.apply(lambda x: target in x['title'], axis=1)]
  return searched

# 검색어와 일치하는 제목만 출력
def search_title_exactly(df, target):
  searched = df[df.apply(lambda x: target == x['title'], axis=1)]
  return searched
```


```python
search_koreauniv = search_title_exactly(sliced_df, "고려대학교")
```


```python
search_sejong = search_title_exactly(sliced_df, "세종(조선)")
```


```python
text = search_koreauniv.iloc[0]['text']
print("text 길이:",len(text))
print(text)
```

    text 길이: 11289
    [include(틀:다른 뜻1, other1=세종특별자치시 조치원읍에 위치한 고려대학교의 분교, rd1=고려대학교/세종캠퍼스)]
    [include(틀:다른 뜻1, other1=KU를 약칭으로 사용하고 있는 다른 대학교, rd1=건국대학교)]
    ||<tablewidth=100%><tablebordercolor=#872434><tablebgcolor=#ffffff,#191919> [[파일:external/upload.wikimedia.org/2000px-Korea_University_Global_Symbol.svg.png|width=20]] {{{#872434,#db7888 '''고려대학교 관련 틀'''}}} ||
    || {{{#!folding [ 펼치기 · 접기 ]
    [include(틀:서울특별시의 대학)]
    [include(틀:서울특별시의 대학별 캠퍼스)]
    [include(틀:서울특별시의 대학별 학부)]
    [include(틀:학교법인 고려중앙학원)]
    [include(틀:대한민국의 소프트웨어중심대학)]
    [include(틀:인공지능대학원협의회)]
    [include(틀:AACSB 국내인증대학)]
    [include(틀:ABEEK 인증대학)]
    [include(틀:KAAB 인증대학)]
    [include(틀:서울총장포럼)]
    [include(틀:APRU 소속 대학)]
    [include(틀:캠퍼스 아시아)]
    }}} ||
    ----
    [include(틀:고려대학교)]
    ----
    ||<-3><tablealign=right><tablebordercolor=#862633><tablebgcolor=#ffffff,#191919><bgcolor=#862633><tablewidth=450> {{{+1 {{{#ffffff '''고려대학교'''}}}}}} [br] {{{#ffffff '''高麗大學校'''}}} [br] {{{#ffcc00 '''KOREA UNIVERSITY'''}}} ||
    ||<-3><bgcolor=#ffffff,#191919> [[파일:external/upload.wikimedia.org/2000px-Korea_University_Global_Symbol.svg.png|width=200]]  ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''교훈'''}}} ||<bgcolor=#ffffff,#191919>{{{#191919,#ffffff '''자유, 정의, 진리'''[br]'''LIBERTAS, JUSTITIA, VERITAS'''}}} ||
    ||<|4><bgcolor=#862633> {{{#ffffff '''상징'''}}} ||<bgcolor=#872434> {{{#ffffff '''동물'''}}} ||{{{#191919,#ffffff   '''호랑이'''}}} ||
    ||<bgcolor=#862633> {{{#ffffff '''교목'''}}} ||{{{#191919,#ffffff  '''잣나무'''}}} ||
    ||<bgcolor=#862633> {{{#ffffff '''교화'''}}} ||{{{#191919,#ffffff  '''철쭉'''}}} ||
    ||<bgcolor=#862633> {{{#ffffff '''교색'''}}} ||[include(틀:표시, other1=#862633, other2=#FFFFFF, other3=크림슨)] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''국가'''}}} ||[[파일:대한민국 국기.svg|width=20px]] [[대한민국]] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''분류'''}}} ||[[파일:external/upload.wikimedia.org/2000px-Korea_University_Global_Symbol.svg.png|width=20]] [[사립대학]] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''개교'''}}} ||[[1905년]] [[5월 5일]] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''설립자'''}}} ||[[이용익]][* [[대한제국]] 시대의 관료, 애국지사, [[독립운동가]]. 자세한 생애는 문서 참조.] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''총장'''}}} ||제20대 [[정진택(대학교수)|정진택]][* 고려대학교 기계공학부에서 학사와 석사 학위, [[미네소타 대학교]]에서 박사 학위를 취득한 이후, 본교 기계공학부 교수로 재직하였다. 고려대학교 공과대학장, 공학대학원장, 기계공학부 학부장, 교수학습개발원장, 대외협력처장 등을 역임하였다.] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''재단 및 법인'''}}} ||학교법인 [[고려중앙학원]] ||
    ||<|1><bgcolor=#862633> {{{#ffffff '''주소'''}}} ||<bgcolor=#862633> {{{#ffffff '''서울'''}}} ||[[서울특별시]] [[성북구]] [[안암로]] 145 ([[안암동|안암동5가]]) ||
    ||<|2><bgcolor=#862633> {{{#ffffff '''재학생'''}}} ||<bgcolor=#862633> {{{#ffffff '''학부생'''}}} ||20,822명{{{-2 (2020년)}}}[* 휴학생 6,961명 미포함] ||
    ||<bgcolor=#862633> {{{#ffffff '''대학원생'''}}} ||8,544명{{{-2 (2020년)}}}[* 휴학생 980명 미포함] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''교직원'''}}} ||4,276명{{{-2 (2020년)}}} ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''국내 분교'''}}} ||[[고려대학교/세종캠퍼스|세종캠퍼스]] ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''대학기본역량진단'''}}} ||자율개선대학{{{-2 (2018년)}}} ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''대학혁신지원사업'''}}} ||A등급{{{-2 (2020년)}}} ||
    ||<-2><bgcolor=#862633> {{{#ffffff '''링크'''}}} ||[[http://www.korea.ac.kr/mbshome/mbs/university/index.do|[[파일:홈페이지 아이콘.png|width=25]]]] [[https://ko-kr.facebook.com/ku1905/|[[파일:페이스북 아이콘.svg|width=25]]]] [[https://www.youtube.com/channel/UCa_Bvlw6AavvDun0_YDcAyQ|[[파일:유튜브 아이콘.svg|height=20]]]] [[https://blog.naver.com/ku_1905|[[파일:네이버 블로그 아이콘.png|width=20]]]] [[https://www.instagram.com/korea_university_official/?hl=ko|[[파일:인스타그램 아이콘.svg|width=25]]]] ||
    [목차]
    [clearfix]
    
    == 개요 ==
    ||<table align=center><tablebordercolor=#872434><tablewidth=90%><bgcolor=#ffffff> {{{#!wiki style="margin: -10px; margin-top: -5px; margin-bottom: -6px"
    [[파일:koreauniversitymaingatemainbuilding.jpg|width=100%]]}}} ||
    ||<bgcolor=#872434> {{{#ffffff '''고려대학교 서울캠퍼스 정문'''}}} ||
    || {{{#!wiki style="margin: -5px -10px"
    [youtube(Hln2_Az_e3k, width=100%)]}}} ||
    ||<bgcolor=#872434> {{{#ffffff '''고려대학교 홍보영상 (2020)'''}}} ||
    || {{{#!wiki style="margin: -5px -10px"
    [youtube(NpW0iz52lVs, width=100%)]}}} ||
    ||<bgcolor=#872434> {{{#ffffff '''고려대학교 홍보영상 ‘Welcome to KU’ (2020)'''}}} ||
    
    [[서울특별시]] [[성북구]] [[안암동]]에 위치한 사립 종합대학교. 
    
    [[1905년]] 충숙공 [[이용익]]이 고종의 지원을 받아 설립한 근대적 사립 고등교육기관인 [[보성전문학교]]에 연원을 두고 있다. 광복 후 [[1946년]] 종합대학으로 승격하며 교명을 고려대학교로 개칭하였다.
    
    약칭은 '''고대(高大)'''이며, [[FM(자기소개)|FM구호]]는 '''민족고대'''[* 1970 민주화 운동 당시 [[서울대학교|서울대]], [[연세대학교|연세대]], [[이화여자대학교|이화여대]]와 [[고려대학교|본교]] 총학생회가 모여 처음으로 '앞으로 이런 일이 재발하지 않도록, 또한 군부 독재 타도와 민주주의의 진전을 위하는 마음으로' 총학생회의 이름 앞에 자주, 민주, 민족, 해방 즉 자주-민주 민족해방 별명을 붙이게 된 것이였고, 각각 [[서울대학교|자주관악]], [[연세대학교|민주연세]], [[고려대학교|민족고대]], [[이화여자대학교|해방이화]]이었다.] 이다.
    
    1980년 [[충청남도]] [[연기군]] [[조치원읍]][* 現 [[세종특별자치시]] [[조치원읍]]]에 [[분교]] 설립을 인가받았다. 자세한 내용은 [[고려대학교 세종캠퍼스]] 참고.
    
    == [[고려대학교/역사|역사]] ==
    [include(틀:고려대학교의 역사)]
    [include(틀:상세 내용, 문서명=고려대학교/역사)]
    
    == 소개 ==
    === 교명 ===
    영문명은 '''Korea University'''이다.
    
    대학의 이름을 '고려(高麗)'로 정한 것은 고려대의 전신인 보성전문학교를 인수한 [[김성수(1891)|인촌 김성수]]의 발상이었는데, 이에 대한 그의 지론은 다음과 같았다.
    
    >우리가 만드는 대학은 반드시 우리나라나 민족을 대표하는 대학이 되도록 하여야 하겠는 만큼 교명도 반드시 그러한 뜻을 나타내는 것이 되어야 하겠는데, 「조선」이나 「한국」[* 여기서의 '한국'은 당시의 역사적 상황이나 문맥상 '대한제국'을 의미하는 것으로 보는 것이 타당할 것이다.]은 역사상 이민족에게 수모를 당한 일이 있어서 싫고, 「고려」도 실은 여진, 몽고 등의 시달림을 받은 일은 있지만 「고구려」의 영광을 계승하여 좋다. 우리나라의 외국어 명칭인 Korea, Corea, Corée도 「고려」의 음을 표기한 것이 아니겠는가. ([[유진오]], <養虎記>. 231쪽-232쪽)
    
    한편 '''고려대학교 정문에는 '고려대학교'라고 적힌 문패가 없다'''. 그 이유가 명확히 알려진 바는 없지만, 굳이 대학의 이름을 적지 않아도 [[http://www.kunews.ac.kr/news/articleView.html?idxno=30353|누구나 이곳이 고려대학교임을 알기 때문]]이라는 말이 전해진다. 이를 소재로 한 [[https://changmoolee.tistory.com/886|시(詩)]]도 존재한다.[* 이 시를 읽으며 서울대학교 관악캠퍼스 정문에도 문패가 없지 않느냐는 의문을 가질 수 있겠으나, 관악캠퍼스로 통합 이전하기 전 서울대학교 대학본부가 위치해 있던 동숭동 캠퍼스의 [[https://www.snu.ac.kr/webdata/old_upload/kor/with_notice/main72.jpg|문리과대학 정문]], [[http://archives.snu.ac.kr/IMAGE/images/M2_1/2017/01/M2_1_12271|법과대학 정문]] 등에는 원래 문패가 있었다. 옛 동숭동 캠퍼스 자리 맞은편의 연건캠퍼스를 꾸준히 지키고 있는 서울대학교 [[https://m.blog.naver.com/donwha88/220890765757?view=img_64|의과대학 정문]]의 경우 정문 자체의 원형을 현재도 잘 간직하고 있다.]
    === 학풍 ===
    [[https://monthly.chosun.com/client/news/viw.asp?ctcd=F&nNewsNumb=201102100023|[집단연구] 고려대 106년의 자화상 : 월간조선]]
    
    '''1. 야성, 저돌성, 중후함, 수수함'''
    고려대학교의 교풍은 야성, 저돌성, 중후함, 수수함 등으로 대표된다. 대학의 상징동물인 호랑이, 이른바 석탑(石塔)으로 일컬어지는 육중한 석조건물, 사실상 대학의 상징주로 여겨지는 막걸리 등 고대를 대표하거나 '고대' 하면 떠오르는 상징들은 대부분  이러한 특징들과 연관된 경우가 많다.
    
    '''2. 협동적, 끈끈함'''
    고려대에서는 졸업생을 '동문', '동창' 등의 단어 대신 '교우'라고 부르는데, 이는 고려대를 같이 다녔다는 이유만으로 친구라는 의미이다. 사회에서 고려대 출신 사이에는 선후배간의 유대가 매우 강한 편이다.
    고대에는 자기 이익만 앞세우려 하기보다는, 타인과 소통하며 서로의 장점을 살려 일을 분담함으로써 시너지를 내는 문화가 발달되어 있다. 또한 일대일 간의 관계보다는 폭넓은 집단주의적 관계를 더 선호하는 편인데[* 흔히 고대는 [[인성]]과 [[인간관계]]를 강조하는 분위기라고 하는데, 여기서 말하는 인성이란 일대일로 짝짓는 관계에서의 배려심이라기보다 전체주의적 [[사회성]]이나 수직적이고 복종적인 [[리더십]]-팔로워십을 의미한다고 보는 게 더 정확하다.], 그렇다 보니 자신과 감정적으로 잘 안 맞는 사람이라 하더라도 [[집단주의|더 큰 조직의 이익을 위해서 조화를 이루고 살아가려 노력]]한다. 구성원들의 애교심이 워낙 커서 그런지, 정치적 이념 및 경제적 이해관계가 다르더라도 같은 고대 동문 사이에는 상대방의 입장에서 상황을 바라보려는 전통이 이어지고 있다.
    
    일례로 고려대는 [[동아리]] 조직이 발달하여, 그 구성원이 인간관계를 다지고 팀플레이를 하는 풍조가 강하다. 공부도 물론 중요시하지만, 개인의 성적만을 챙기는 능력보다는 사회 속의 인간관계를 관리하는 능력, 남을 복종시키고 상급자에게 복종하는 지도력, 또는 친화력 등을 더 높이 평가한다. 다른 그 무엇보다도 장기적인 대인관계와 신뢰감을 중시하는 습관, [[총대#s-3|총대 메는]] 일을 두려워 하지 않는 기질이 이런 문화 속에서 길러지는 건 당연한 일이다. 과거에 '고대는 집단주의, 연대는 개인주의'라는 말이 있었던 것도 이러한 이유라고 볼 수 있다. 다만 요즘은 서울 주요 명문학군 내지 명문고 출신자가 많은 것 때문인지, 아니면 학생들의 기질이 많이 희석되었는지 고대생들도 다른 대학에 비해선 여전히 전체주의적이나 개인주의적 면모가 매우 강해졌다고 느껴진다.
    
    '''3. 개방적, 포용적'''
    사실 고대생의 끈끈한 이미지를 생각한다면 외부에 대해 배타적이고 폐쇄적일 것처럼 생각하기 쉽다. 하지만 고려대 교수들 가운데 자교 출신 비율이 60% 정도에 불과한 것[* 고려대학교의 역사를 수놓은 수많은 교수들 가운데 다수의 교수들, 예를 들어 [[김병로]], [[현상윤]], [[안호상]], [[오천석]], [[유진오]], [[손진태]], [[이상은(학자)|이상은]], [[김상협]], [[김준엽]], [[현승종]], [[윤천주]], 김충렬, [[김우창]], [[문국진]], 이필상, 김화영 등이 타교 출신이다.], 타 학부 출신 고려대 대학원생에 대한 대우 등에서 알 수 있듯이 고려대는 사실 이러한 부분에서 매우 개방적인 학교이다. [* 타 학부 출신 편입생이나 대학원생이라 하더라도 뭔가 능력을 갖고 있다 하면 주류로 편입시켜준다. 대표적인 인물로는 외대에서 고법으로 편입한 오세훈 전 서울시장.]
    
    더불어, 각 교수 및 학생들의 특기와 전문성 등을 존중하는 경향이 있다. 어느 사회에서든 조직이나 단체가 크게 발달하기 위해서는 그 모임 내부에서 독특하고 새로운 시도를 하는 것이 필요한데, 그러한 시도가 고려대 특유의 상술한 분위기에 의해 힘을 받아, 지금처럼 구성원의 도전정신을 장려하는 문화가 만들어진 것이다. 
    
    '''4. 집념'''
    연구에 있어서는 특유의 집념과 저력으로 장기간의 꾸준한 연구를 요하는 분야에서 두각을 나타낸다. 이는 본교 특유의 집단적 역할분담 문화가, 애매한 '멀티 플레이어' 또는 '제너럴리스트'보다는, 확고한 '스페셜리스트'를 더 선호하기 때문에 그러하다고 보여진다. 물론 ‘석탑’으로 대표되는 본교 동문의 '불굴의 기질' 또한 무시할 수 없다. 그래서인지 고대 출신 인물들은 날렵함, 또는 눈치 싸움으로 승부하는 분야보다는 지구력이나 참을성, 우직함으로 승부를 보는 분야에서 매우 강하다. 예를 들어 본교가 자랑하는 [[법학]]은 장기간의 지루한 공부를 견뎌내야 하는 분야이므로, 강세를 나타내 온 것이다.
    
    '''5. 저항정신'''
    고대의 학문적 기조는, 기성 학문의 대세를 따르기보다는 독자적 대안을 제시하려는 경향이 강하다. 데이터분석을 통한 수리논증이 대세가 될 때에 그에 맞서 이론분석의 방법론을 동등하게 강조하기도 했고, 미국/일본 유학파가 주류를 이룰 때에는 그에 맞서 영국, 프랑스, 독일 등의 학문을 적극적으로 도입하고 소개하기도 했다. 더불어 미국이나 일본에서 새로운 사조가 들어와서 우리 학계 전체를 휩쓸 경우에도 거기에 맹목적으로 따르지 않고, 전통적, 기본적, 원칙적인 입장을 고수하였다.
    일제 시절에는 일본문화가 워낙 주류를 차지하다보니 학문에 있어서도 [[민족주의]]적 경향이 매우 강했다. 사학과의 경우, 제국대학시절 영향으로 실증사관 중심의 논증을 중시하는 서울대의 가장 대척점에 서는 대학이 고려대이다. 참여정부가 추진한 한일공동역사연구회의 주축은 고려대 출신과 고려대 교수들인 [[조광]] [[김현구]] 조법종 등 이었고, 식민사관에 대해 자주 언급하고 비판하는 [[이희진]] 교수 또한 고려대 사학과 출신이다. 임나일본부설에 가장 실랄히 비판을 가하는 학자중 한명인 이재석 교수 또한 고대 학부 출신이다. 한편 민족주의의 병폐가 오히려 두드러진 이후에는 민족주의적 경향을 차차 희석시키기도 하였다. 물론 사이비 까지는 아니지만 이희진 교수나 최재석 교수 같은 경우는 다소 무리한 주장을 많이 하기 때문에 비판 받아왔다. 일례로 해방 이후 [[한글전용]]운동이 큰 흐름을 타자, 고대는 이에 반대했다.[* 연세대에서 외솔 [[최현배]] 교수가 순우리말 쓰기 운동을 주도하였을 때, --순우리말에서 순(純)은 그럼 한자(漢字)가 아니라는 건가?-- 고려대에서 중국철학 및 유학사를 가르쳤던 경락 [[이상은(학자)|이상은]] 교수가 그에 대해 반대한 얘기는 유명하다.] 그리고 대한민국 교육이 한문을 점점 소홀히 하기 시작할 때 고대는 오히려 학생들의 [[한문]] 실력을 대단히 중시하였고, 이는 오늘날까지 교내 졸업요건에 한자 급수를 포함시킴으로써 이어 오고 있다. 이런 이유로 한때 본교의 학풍이 '보수적'이라는 오해를 사기도 했으나,이는 오히려 새로운 대세에 쉽게 휩쓸리지 않는 당당하고 굳건한 기질로 재평가할 필요가 있다.
    
    위의 사실들을 종합해 볼 때, 현재 고려대의 학풍은 자유로우며 개혁적이고 진보적인 스탠스에 한국 특유의 정(情)이 합쳐져 지금의 모습을 갖추게 되었다고 볼 수 있겠다.
    
    하지만 이와 별개로 실제 대학 생활에서는 여전히 수직적이고 가부장적인 문화가 많다. 실제로 동문 모임이나 학교 생활 대부분에서 '고대인다운 모습'이 강요된다. 더하여 학생들에 비해 학교를 구성하는 교직원, 또는 운영주체의 스탠스가 상당히 보수적인 면이 있으며, 이것의 예로 모바일 학생증 및 안드로이드 애플리케이션의 제작 규제를 들 수 있다. 
    
    == [[학교법인 고려중앙학원|재단]] ==
    [include(틀:상세 내용, 문서명=학교법인 고려중앙학원)]
    
    == [[고려대학교/상징|상징]] ==
    [include(틀:상세 내용, 문서명=고려대학교/상징)]
    
    == [[고려대학교/학생운동|학생운동]] ==
    [include(틀:상세 내용, 문서명=고려대학교/학생운동)]
    
    == [[고려대학교/교우회|교우회]] ==
    [include(틀:상세 내용, 문서명=고려대학교/교우회)]
    
    == [[고려대학교/학사제도|학사제도]] ==
    [include(틀:상세 내용, 문서명=고려대학교/학사제도)]
    == [[고려대학교/강의|강의]] ==
    [include(틀:상세 내용, 문서명=고려대학교/강의)]
    
    == [[고려대학교/학부|학부]] ==
    [include(틀:상세 내용, 문서명=고려대학교/학부)]
    
    == [[고려대학교/대학원|대학원]] ==
    [include(틀:상세 내용, 문서명=고려대학교/대학원)]
    
    == [[고려대학교/총학생회|총학생회]] ==
    [include(틀:상세 내용, 문서명=고려대학교/총학생회)]
    
    == [[고려대학교/동아리|동아리]] ==
    [include(틀:상세 내용, 문서명=고려대학교/동아리)]
    
    == [[고려대학교/시설|시설]] ==
    [include(틀:상세 내용, 문서명=고려대학교/시설)]
    
    === [[고려대학교/식당 및 매점|식당 및 매점]] ===
    [include(틀:상세 내용, 문서명=고려대학교/식당 및 매점)]
    
    == [[고려대학교의료원]] ==
    [include(틀:상세 내용, 문서명=고려대학교의료원)]
    
    == [[고려대학교/교통|캠퍼스의 교통]] ==
    [include(틀:상세 내용, 문서명=고려대학교/교통)]
    [include(틀:상세 내용, 문서명=고려대학교/세종캠퍼스)]
    
    == [[고려대학교/행사|각종 행사]] ==
    [include(틀:상세 내용, 문서명=고려대학교/행사)]
    
    == [[고려대학교/사건사고|사건사고]] ==
    [include(틀:상세 내용, 문서명=고려대학교/사건사고)]
    
    == 노동조합 현황 ==
     * [[전국대학노동조합]] 고려대학교지부: [[민주노총]] 소속.
     * [[전국대학노동조합]] 고려대학교2지부: [[민주노총]] 소속.
    
    == [[고려대학교/출신 인물|출신 인물]] ==
    [include(틀:상세 내용, 문서명=고려대학교/출신 인물)]
    
    == 관련 문서 ==
     * [[고려대학교/역사]]
     * [[고려대학교/상징]]
     * [[고려대학교/학생운동]]
     * [[고려대학교/학부]]
     * [[고려대학교/대학원]]
     * [[고려대학교/시설]]
     * [[고려대학교/식당 및 매점]]
     * [[고려대학교/주변 상권]]
     * [[고려대학교/총학생회]]
     * [[고려대학교/동아리]]
     * [[고려대학교/언론 및 커뮤니티]]
     * [[고려대학교/교통]]
     * [[고려대학교/사발식]]
     * [[고려대학교/응원가]]
     * [[고려대학교/기타 정보]]
     * [[고려대학교/출신 인물]]
     * [[고려대학교/교우회]]
     * [[고려대학교/세종캠퍼스]]
     * [[고려대학교의료원]]
     * [[고대빵]]
     * [[고연전]]
     * [[고파스]]
     * [[쿠플존]]
     * [[막걸리 찬가]]
     * [[보성전문학교]]
     * [[안암역]], [[고려대역]]
     * [[김성수(1891)|김성수]] - 고려대학교의 전신인 [[보성전문학교]]를 인수
     * [[이용익]] - 고려대학교의 전신인 [[보성전문학교]]의 설립자
     * [[4.18 의거]]
     * [[고전음악감상실]]
    
    [[분류:고려대학교]]
    

문서 길이가 길면 아래처럼 indexerror를 낸다.


```python
print("원문 text 길이:",len(text))

raw_input_ids = tokenizer.encode(text) # 토큰화, 정수 인코딩
input_ids = [tokenizer.bos_token_id] + raw_input_ids + [tokenizer.eos_token_id] # bos, eos 토큰 추가
summary_ids = model.generate(torch.tensor([input_ids]),  num_beams=4,  max_length=512,  eos_token_id=1)
result = tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
print("요약문:",result)
print("요약문 text 길이:",len(result))
```

    원문 text 길이: 11289
    


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-26-64829a7a1994> in <module>
          3 raw_input_ids = tokenizer.encode(text) # 토큰화, 정수 인코딩
          4 input_ids = [tokenizer.bos_token_id] + raw_input_ids + [tokenizer.eos_token_id] # bos, eos 토큰 추가
    ----> 5 summary_ids = model.generate(torch.tensor([input_ids]),  num_beams=4,  max_length=512,  eos_token_id=1)
          6 result = tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
          7 print("요약문:",result)
    

    /usr/local/lib/python3.8/dist-packages/torch/autograd/grad_mode.py in decorate_context(*args, **kwargs)
         25         def decorate_context(*args, **kwargs):
         26             with self.clone():
    ---> 27                 return func(*args, **kwargs)
         28         return cast(F, decorate_context)
         29 
    

    /usr/local/lib/python3.8/dist-packages/transformers/generation/utils.py in generate(self, inputs, generation_config, logits_processor, stopping_criteria, prefix_allowed_tokens_fn, synced_gpus, **kwargs)
       1250             # if model is encoder decoder encoder_outputs are created
       1251             # and added to `model_kwargs`
    -> 1252             model_kwargs = self._prepare_encoder_decoder_kwargs_for_generation(
       1253                 inputs_tensor, model_kwargs, model_input_name
       1254             )
    

    /usr/local/lib/python3.8/dist-packages/transformers/generation/utils.py in _prepare_encoder_decoder_kwargs_for_generation(self, inputs_tensor, model_kwargs, model_input_name)
        615         encoder_kwargs["return_dict"] = True
        616         encoder_kwargs[model_input_name] = inputs_tensor
    --> 617         model_kwargs["encoder_outputs"]: ModelOutput = encoder(**encoder_kwargs)
        618 
        619         return model_kwargs
    

    /usr/local/lib/python3.8/dist-packages/torch/nn/modules/module.py in _call_impl(self, *input, **kwargs)
       1192         if not (self._backward_hooks or self._forward_hooks or self._forward_pre_hooks or _global_backward_hooks
       1193                 or _global_forward_hooks or _global_forward_pre_hooks):
    -> 1194             return forward_call(*input, **kwargs)
       1195         # Do not call functions when jit is used
       1196         full_backward_hooks, non_full_backward_hooks = [], []
    

    /usr/local/lib/python3.8/dist-packages/transformers/models/bart/modeling_bart.py in forward(self, input_ids, attention_mask, head_mask, inputs_embeds, output_attentions, output_hidden_states, return_dict)
        808             inputs_embeds = self.embed_tokens(input_ids) * self.embed_scale
        809 
    --> 810         embed_pos = self.embed_positions(input)
        811         embed_pos = embed_pos.to(inputs_embeds.device)
        812 
    

    /usr/local/lib/python3.8/dist-packages/torch/nn/modules/module.py in _call_impl(self, *input, **kwargs)
       1192         if not (self._backward_hooks or self._forward_hooks or self._forward_pre_hooks or _global_backward_hooks
       1193                 or _global_forward_hooks or _global_forward_pre_hooks):
    -> 1194             return forward_call(*input, **kwargs)
       1195         # Do not call functions when jit is used
       1196         full_backward_hooks, non_full_backward_hooks = [], []
    

    /usr/local/lib/python3.8/dist-packages/transformers/models/bart/modeling_bart.py in forward(self, input_ids, past_key_values_length)
        136         ).expand(bsz, -1)
        137 
    --> 138         return super().forward(positions + self.offset)
        139 
        140 
    

    /usr/local/lib/python3.8/dist-packages/torch/nn/modules/sparse.py in forward(self, input)
        158 
        159     def forward(self, input: Tensor) -> Tensor:
    --> 160         return F.embedding(
        161             input, self.weight, self.padding_idx, self.max_norm,
        162             self.norm_type, self.scale_grad_by_freq, self.sparse)
    

    /usr/local/lib/python3.8/dist-packages/torch/nn/functional.py in embedding(input, weight, padding_idx, max_norm, norm_type, scale_grad_by_freq, sparse)
       2208         # remove once script supports set_grad_enabled
       2209         _no_grad_embedding_renorm_(weight, input, max_norm, norm_type)
    -> 2210     return torch.embedding(weight, input, padding_idx, scale_grad_by_freq, sparse)
       2211 
       2212 
    

    IndexError: index out of range in self


<br>

## 3. 특수문자, 기호 지우기


```python
def preprocess(text):
  preprocessed = re.sub('[^가-힣0-9]', ' ', text) # 한글, 숫자만 남기기. 영어는 본문이 아닌 불필요한 소모가 너무 큼
  nonblank = ' '.join(preprocessed.split()) # 공백 중복된거 1개로
  return nonblank
```


```python
text = sliced_df.iloc[0]['text']
preprocessed_text = preprocess(text)
print("전처리 후:",preprocessed_text)
```

    전처리 후: 목차 1 개요 파일 3444050440 60 신 세계수의 미궁 2 파프니르기사 신 세계수의 미궁 2 에서 뜬 아앗 세계수의 미궁 시리즈 에 전통으로 등장하는 대사 세계수의 미궁 2 제왕의 성배 2편 부터 등장했으며 훌륭한 사망 플래그 의 예시이다 세계수의 모험가들이 탐험하는 던전인 수해의 구석구석에는 채취 벌채 채굴 포인트가 있으며 이를 위한 채집 스킬에 투자하면 제한된 채집 기회에서 보다 큰 이득을 챙길 수 있다 그러나 분배할 수 있는 스킬 포인트는 한정되어 있기 때문에 채집 스킬에 투자하는 만큼 전투 스킬 레벨은 낮아지게 된다 다만 채집 시스템은 신 세계수 시리즈의 그리모어 복제 복합 채집 스킬인 야생의 감 5편의 종족 특유 스킬 크로스의 1레벨이 만렙인 채집 스킬 등으로 편의성이 점차 나아져서 채집 스킬 때문에 스킬 트리가 내려가는 일은 점점 줄어들었다 아앗 이 발생하는 과정을 요약하면 다음과 같다 1 채집용 캐릭터들로 이루어진 약한 파티 레인저 세계수의 미궁 2 레인저 5명 가 수해에 입장한다 1 필드 전투를 피해 채집 포인트에 도착한 후 열심히 아이템을 캐는 중에 1 아앗 라플레시아가 나타났다 이때 등장하는 것은 세계수의 미궁 시리즈 는 아니지만 훨씬 위층에 등장하는 강력한 필드 몬스터이며 선제 공격을 당하게 된다 1 으앙 죽음 여담으로 아앗 의 유래는 1인칭 던전 크롤러의 원조 위저드리 에서 함정을 건드렸을 때 나오는 대사 라고 한다 각 작품에서의 모습 세계수의 미궁 2 제왕의 성배 아앗 의 악랄함은 첫 등장한 작품이자 시리즈 중에서도 불친절하기로 정평이 난 2편이 절정이었다 그야말로 위의 아앗 시퀀스 그대로 묻지도 따지지도 않고 채집할 때마다 일정 확률로 강제로 전투에 돌입해야 했다 게다가 이럴 때 쓰라고 있는 레인저의 스킬 위험 감지 중간 확률로 적의 선제 공격을 무효화 는 정작 작동하지 않는다 참고로 2편에서 채집 도중 아앗 이 뜰 확률은 910 고작 1 다 던파확률의 법칙 낮아 보이는 확률이어도 플레이 중 한 번이라도 일어나는 것 을 경험하는 체감 확률을 고려하여 확률을 설정한다고 세계수의 미궁 3 성해의 내방자 다행히 채집 중 낮은 확률로 좋은 아이템을 얻을 수 있을 것 같지만 주변에서 몬스터들의 기척이 느껴진다 는 메시지가 뜨고 이때 운이 좋으면 레어 아이템을 얻을 수 있지만 반대의 경우 적과 싸우게 되는 것으로 조정되었다 세계수의 미궁 4 전승의 거신 기본적인 것은 3편과 같지만 4편에서는 움직이지 않고 채집할 때도 턴이 경과하도록 조정되었기 때문에 주변에 있는 를 잊고 채집에 몰두하다가 와 부딪히면 버전 아앗 이 뜬다 그리고 난이도 로 플레이시 로 인한 아앗 을 제외하면 절대로 발생하지 않는다 신 세계수의 미궁 밀레니엄의 소녀 신 세계수의 신 세계수의 미궁 2 파프니르기사 미궁 시리즈 채집 방식이 한 턴으로 끝나는 구조 채집으로 한 번 아이템을 획득하면 다시 채집 스킬 에 의해 가 뜨면서 한꺼번에 획득되는 구조 로 바뀐 덕분인지 강제 조우로 다시 회귀해버렸다 그나마 위험 감지 먹통과 같은 버그성 난점들은 수정되었다 그 이후에 나온 세계수의 미궁 5 오랜 신화의 끝 과 시리즈의 집대성 작품이자 3 마지막 작품인 세계수의 미궁 도 마찬가지 세계수의 미궁 본작의 채집은 신 세계수 시리즈와 같은 매커니즘이라 굳이 언급할 필요는 없으나 퀘스트중에 2편의 아앗 시퀀스를 재현하면서 라플레시아 가 등장하는 퀘스트가 존재한다 깨알같이 시스템 메세지 창이 아니라 대화창을 이용해서 완벽 재현한 것이 포인트 페르소나 섀도우 오브 더 래버린스 세계수 시스템을 기반으로 한 페르소나 시리즈 와의 콜라보 작품인 페르소나 에서도 등장한다 3 4편과 같이 파워 스폿에서 채집 도중 메시지가 뜨며 실패하면 파티에 참가하고 있는 멤버 중 한 명의 25683358 아앗 하는 음성 또는 코로마루 개소리 과 함께 그 던전의 강적 인 거대 섀도 페르소나 시리즈 섀도우 가 나타난다 그러나 내비 전용 스킬인 뱀눈 노려보기 위험 감지와 같은 효과 와 채집 보조 스킬은 파티의 전투력에 전혀 지장을 주지 않으며 대안심 을 달면 거의 볼 일이 없어져서 초중반 이후에는 존재감이 급격히 줄어든다 분류 세계수의 미궁 시리즈
    


```python
def preprocessed_list():
  result = []
  for i in range(sliced_df.shape[0]):
    text = sliced_df.iloc[i]['text']
    preprocessed = preprocess(text)
    result.append(preprocessed)
  return result
```


```python
result = preprocessed_list()
```

불용어 후보

'include 틀', 'tablewidth', 'tablebordercolor', 'wiki', 'tablealign', 'tablebordercolor', 'background'


근데 이 많은 데이터를 다 토큰화해서 불용어 처리 하는 건 사실상 불가능해보임.

<br>

## 4. 데이터 csv로 저장

만들어진 데이터프레임을 원하는 디렉토리에 저장


```python
%cd /content/drive/My Drive/kubig/kucon
```

    /content/drive/My Drive/kubig/kucon
    


```python
sliced_df.to_csv('namuwiki_df.csv', encoding='utf-8', index=False)
```


```python
new_df = pd.read_csv('/content/drive/MyDrive/kubig/kucon/namuwiki_df.csv')
```
<br>

## 5. 쟁점

1) 길이가 너무 짧은 문서들은 어떻게 할 것인가? -> 기준(250자)를 정해두고 그에 못 미치는 문서는 제거

2) 길이가 너무 긴 문서들은 indexerror가 뜨는데, 어떻게 할 것인가? -> 문서의 특정 부분만 갖고와서 요약해야 하나?

3) 나무위키 특성상 도표, 링크, 인용구 등에 소비되는 특수문자가 아주 많다. 이를 어떻게 전처리 할 것인가?

<br>
