---
title: "[NLP] Word2Vec 연습"
excerpt: "Word2Vec를 사용하여 네이버 영화 리뷰 단어 토큰화"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, 추천시스템, TF-IDF]

published: true

categories:
  - DL

date: 2023-02-01 22:00:00
last_modified_at: 2023-02-01 22:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>


<br>

## 1. import, load data


```python
!pip install konlpy
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting konlpy
      Downloading konlpy-0.6.0-py2.py3-none-any.whl (19.4 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m19.4/19.4 MB[0m [31m76.7 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting JPype1>=0.7.0
      Downloading JPype1-1.4.1-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (465 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m465.6/465.6 KB[0m [31m40.3 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: lxml>=4.1.0 in /usr/local/lib/python3.8/dist-packages (from konlpy) (4.9.2)
    Requirement already satisfied: numpy>=1.6 in /usr/local/lib/python3.8/dist-packages (from konlpy) (1.21.6)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from JPype1>=0.7.0->konlpy) (21.3)
    Requirement already satisfied: pyparsing!=3.0.5,>=2.0.2 in /usr/local/lib/python3.8/dist-packages (from packaging->JPype1>=0.7.0->konlpy) (3.0.9)
    Installing collected packages: JPype1, konlpy
    Successfully installed JPype1-1.4.1 konlpy-0.6.0
    


```python
from google import colab
colab.drive.mount("/content/drive")
```

    Mounted at /content/drive
    


```python
import pandas as pd
from glob import glob
import os
import numpy as np
from tqdm import tqdm, tqdm_notebook
import re

from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity 
from konlpy.tag import Okt
```


```python
df = pd.read_csv("./drive/MyDrive/kubig/mangoplate.csv")
```

    /usr/local/lib/python3.8/dist-packages/IPython/core/interactiveshell.py:3326: DtypeWarning: Columns (4) have mixed types.Specify dtype option on import or set low_memory=False.
      exec(code_obj, self.user_global_ns, self.user_ns)
    

<br>

## 2. 데이터 살피기


```python
df.head()
```





  <div id="df-58ff216d-8b88-4544-a6cd-3a0947dd6ca9">
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
      <th>review</th>
      <th>taste</th>
      <th>title</th>
      <th>가고싶다</th>
      <th>전체평점</th>
      <th>주소</th>
      <th>음식종류</th>
      <th>locate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>안국에서 제일 좋아하는 곳이에요!\n여기 정말 분위기가 좋아요~ 한옥을 개조해서 만...</td>
      <td>맛있다</td>
      <td>만가타</td>
      <td>2,799</td>
      <td>4.8</td>
      <td>서울시 종로구 소격동 88-17</td>
      <td>세계음식 기타</td>
      <td>북촌</td>
    </tr>
    <tr>
      <th>1</th>
      <td>간만에 100단위 리뷰는 정말 오랜만이었던 게더링! 예전에 밋업 열렸을 때 부터 스...</td>
      <td>맛있다</td>
      <td>만가타</td>
      <td>2,799</td>
      <td>4.8</td>
      <td>서울시 종로구 소격동 88-17</td>
      <td>세계음식 기타</td>
      <td>북촌</td>
    </tr>
    <tr>
      <th>2</th>
      <td>너의 모든 것을 사랑할 것만가타\n\n아니, 이미 만가타의 모든 것을 사랑하게 됐다...</td>
      <td>맛있다</td>
      <td>만가타</td>
      <td>2,799</td>
      <td>4.8</td>
      <td>서울시 종로구 소격동 88-17</td>
      <td>세계음식 기타</td>
      <td>북촌</td>
    </tr>
    <tr>
      <th>3</th>
      <td>*미트볼, 오픈샌드위치 2종류, 오리리조또x2 (4인)\n새우들은 샌드위치가 맛있었...</td>
      <td>맛있다</td>
      <td>만가타</td>
      <td>2,799</td>
      <td>4.8</td>
      <td>서울시 종로구 소격동 88-17</td>
      <td>세계음식 기타</td>
      <td>북촌</td>
    </tr>
    <tr>
      <th>4</th>
      <td>가을쯤 방문했던 만가타를 이제야 쓰다니...ㅠㅠ\n\n만가타는 이때까지 갔던 레스토...</td>
      <td>맛있다</td>
      <td>만가타</td>
      <td>2,799</td>
      <td>4.8</td>
      <td>서울시 종로구 소격동 88-17</td>
      <td>세계음식 기타</td>
      <td>북촌</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-58ff216d-8b88-4544-a6cd-3a0947dd6ca9')"
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
          document.querySelector('#df-58ff216d-8b88-4544-a6cd-3a0947dd6ca9 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-58ff216d-8b88-4544-a6cd-3a0947dd6ca9');
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
df.info() # review와 전체평점에 결측치 있는 것으로 확인
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 98373 entries, 0 to 98372
    Data columns (total 8 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   review  98021 non-null  object
     1   taste   98373 non-null  object
     2   title   98373 non-null  object
     3   가고싶다    98373 non-null  object
     4   전체평점    98244 non-null  object
     5   주소      98373 non-null  object
     6   음식종류    98373 non-null  object
     7   locate  98373 non-null  object
    dtypes: object(8)
    memory usage: 6.0+ MB
    


```python
print("식당 개수:",len(df['title'].unique()))
```

    식당 개수: 1759

<br>
    

## 3. 데이터프레임 전처리

코사인 유사도를 분석하기에 앞서 식당을 기준으로 review를 통합하는 작업을 해야한다고 생각했다. 그러지 않는 경우 하나의 식당을 입력했을 때 출력에 동일한 식당이 중복되어 등장할 수도 있기 때문이다.

우선 review에서 결측치가 있는 row를 제거한다.


```python
df = df[~df['review'].isnull()]
```

각 가게별로 review가 몇 개 있는지 세어보도록 했다. 그리고 리뷰 개수를 cnt라는 컬럼으로 만들어 기존 데이터에 병합해준다.


```python
cnt_title = pd.DataFrame(df.groupby('title')['title'].count().sort_values(ascending=False))
cnt_title.columns = ['cnt']
cnt_title.reset_index(inplace=True)
merge_df = pd.merge(df, cnt_title, on='title')
```

리뷰가 5개 미만인 식당은 추천 대상으로 보기에 어려움이 있으니, 이런 식당은 없애기로 한다.


```python
merge_df.drop(merge_df[merge_df['cnt']<5].index, inplace=True)
```

식당 이름을 기준으로 groupby해서 review들을 이어붙여준다(sum).


```python
review_concat_df = pd.DataFrame(merge_df.groupby('title')['review'].sum())
review_concat_df.reset_index(inplace=True)
```

<br>

## 4. review 전처리

불용어로 쓸 txt 파일을 미리 디렉토리에 저장해놓고 불러온다.


```python
with open('/content/drive/MyDrive/sentence/stopwords.txt',  encoding='cp949') as f:
    list_file = f.readlines()
    stopwords = list_file[0].split(",")

okt = Okt()
```

`expected string or bytes-like object`라는 type error가 등장할 때는 정규표현식에서 에러가 난 경우이다. 이유는 탐색 대상이 되는 data 부분이 문자열이 아니기 때문이다. 따라서 str로 감싸면 해결된다.


```python
def review_preprocess(data):
  data = re.sub('[^가-힣]', ' ', str(data)) # 정규표현식으로 한글만 남기기
  data = okt.morphs(data, stem=True) # 형태소 단위로 토큰화
  data = [x for x in data if x not in stopwords] # 불용어 제거
  data = " ".join(data) # 한 문장으로 이어붙이기
  return data
```


```python
review_concat_df['prepro_review'] = review_concat_df['review'].map(lambda x:review_preprocess(x))
```


```python
review_concat_df.head()
```





  <div id="df-2d30a53c-b2be-43bf-a038-37111335d3c6">
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
      <th>review</th>
      <th>prepro_review</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>128Pan</td>
      <td>\n          128토마토스튜. 풍기리조또. 당일(일욜)에 예약 문의를 조심...</td>
      <td>토마토 스튜 풍기 리조또 당일 일욜 예약 문의 조심 스럽게 드디어 먹다 맛있다 빵 ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>129 라멘하우스</td>
      <td>\n          4월 16일 방문, 오픈 시간보다 약간 늦게 왔는데 다 먹고 ...</td>
      <td>방문 오픈 늦다 다 먹다 나가다 쯤 재료 소진 대서 인테리어 라멘 집 별로 어울리다...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>136길육미</td>
      <td>\n          민물장어솥밥은 너무너무 맛있었어요! 맨치까스는 쏘쏘 ㅠㅠ 밖에...</td>
      <td>민물장어 솥밥 맛있다 매다 스 쏘다 쏘다 밖 매장 작다 줄 안 들어가다 층 사람 북...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17도씨</td>
      <td>가벼운 마음 전하기엔, 연남동 &lt;17도씨&gt;\n랍상소총과 패션후르츠 초콜릿 내 취향\...</td>
      <td>가볍다 마음 전 연남동 도씨 랍 상 소총 패션후르츠 초콜릿 내 취향 칼로리 크기 사...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1842</td>
      <td>\n          여행 다녀온지 1년이 다되도록 잊혀지지 않는 체코 요리 스비치...</td>
      <td>여행 다녀오다 다 잊혀지다 체코 요리 스 비치 코바 이름 저렇게 어렵다 잊혀지다 진...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-2d30a53c-b2be-43bf-a038-37111335d3c6')"
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
          document.querySelector('#df-2d30a53c-b2be-43bf-a038-37111335d3c6 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-2d30a53c-b2be-43bf-a038-37111335d3c6');
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

## 5. TF-IDF

TF-IDF를 만들게 될 경우 단어의 수가 몇 개까지 늘어나는지 파악하기 위해 아무 파라미터도 설정하지 않고 차원을 출력해보자.


```python
tfidf = TfidfVectorizer()
review_tfidf = tfidf.fit_transform(review_concat_df['prepro_review'])
print(review_tfidf.shape) 
```

    (1724, 49056)
    

5만 개에 육박하는 단어가 너무 많아서 sparse하다 싶으면 이를 제어해주는 파라미터가 `max_features`이다. 10000으로 설정하면 단어 수가 10000개로 줄어든다. 또한 `min_df`는 최소 몇개 이상의 문서(이 경우엔 review)에서 등장한 단어만을 추릴 것인지 지정한다. 2로 지정하면 1개의 review에만 등장한 단어는 제거된다.


```python
tfidf = TfidfVectorizer(min_df = 2, analyzer='word', max_features=30000)

review_tfidf = tfidf.fit_transform(review_concat_df['prepro_review'])
review_similar = cosine_similarity(review_tfidf, review_tfidf)
```

<br>

## 6. 유사도 높은 맛집 출력!


```python
# id와 식당 이름을 매핑할 dictionary를 생성
place2id = {}
for i, c in enumerate(review_concat_df['title']): 
  place2id[i] = c

id2place = {}
for i, c in place2id.items(): 
  id2place[c] = i
```


```python
def recommend_place(place):
  idx = id2place[place] # 원하는 식당의 인덱스를 저장
  sim_scores = [(i, c) for i, c in enumerate(review_similar[idx]) if i != idx] # 코사인 유사도에서 원하는 식당의 row만 추출
  sim_scores = sorted(sim_scores, key = lambda x: x[1], reverse=True) # 추출한 row에서 유사도가 높은 순으로 정렬
  sim_scores = [(place2id[i], score) for i, score in sim_scores[0:10]] # 10위까지 자르기
  return sim_scores
```


```python
recommend_place('송포갈비') # 갈비
```




    [('청죽골식당', 0.6530340600878778),
     ('무학', 0.6220287940091977),
     ('노란상소갈비', 0.6104216045046182),
     ('장수갈비', 0.5098244962558016),
     ('양식당더램키친', 0.5001685296544661),
     ('조선옥', 0.49228278715332474),
     ('호남식당', 0.4731945281831394),
     ('통의동 국빈관', 0.45635287925757095),
     ('백송', 0.39240008274271687),
     ('이치류', 0.3813179355563731)]




```python
recommend_place('스시산') # 초밥
```




    [('스시키', 0.8186678279704203),
     ('스시소라', 0.7558051889091664),
     ('이요이요스시', 0.7502284017791456),
     ('스시고', 0.7474935235655275),
     ('김수사', 0.7460174705084621),
     ('하쯔호', 0.7427481703016261),
     ('스시만', 0.731894446758999),
     ('스시조', 0.7284070761139728),
     ('타쿠미곤', 0.7279397913282535),
     ('스시타노', 0.7276777446256351)]




```python
recommend_place('방배목장') # 아이스크림/카페
```




    [('삼청동쿠크', 0.4639018989307276),
     ('크레마디몬타냐', 0.4027597012256022),
     ('오드펠로우즈', 0.37499058095365384),
     ('솔티밥', 0.35539610231364),
     ('서울커피', 0.30190013112753816),
     ('카페로우슬로우', 0.2975239202220727),
     ('당도', 0.28582445839608134),
     ('카페브르브르', 0.24771532146653125),
     ('솔리드웍스', 0.2412696923080974),
     ('비하인드리메인', 0.23465655375335046)]


<br>

## 7. 느낀 것

 - 어느 식당은 리뷰가 수백 개에 달하고 어느 식당은 10개 미만이다. 식당마다 리뷰 수가 다른 것을 어떻게 보정해줄지 고민해볼 만하다. 식당마다 리뷰 개수를 통일하는 방법도 있고, 통일한다면 몇 개로 할지도 고민해봐야 한다.
 - 리뷰 뿐만이 아니라 평점, 위치, 음식 종류에 따른 유사도도 추가해야 제대로 된 추천 시스템이 구현될 것 같다.
 - 다만 여러 유사도를 결국 하나로 합치는 과정이 필요할 것 같은데, 각각에 얼만큼 가중치를 부여하여 합칠 것인지 명확한 기준을 세우기가 어려울 것 같다.
 - 위치 기반 유사도를 판단하는 것은 위도,경도 데이터를 불러와서 잘 매핑해보면 될 것 같은데, 나중에 구현해보면 재밌을 것 같다.
 - 실제로 쓰이는 식당 추천 알고리즘 원리들을 엿볼 수 있다면 흥미로울 것 같다.
