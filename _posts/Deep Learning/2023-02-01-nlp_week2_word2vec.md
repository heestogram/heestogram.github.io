---
title: "[NLP] Word2Vec 연습"
excerpt: "Word2Vec를 사용하여 네이버 영화 리뷰 단어 토큰화"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, Word2Vec]

published: true

categories:
  - DL

date: 2023-02-01 21:00:00
last_modified_at: 2023-02-01 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>


```python
import gensim
gensim.__version__
```




    '3.6.0'




```python
!pip install konlpy
```


### Okt 느릴 경우 Mecab 사용!
Okt 형태소 분석기가 느릴 경우 Mecab으로 하면 더 나은 속도로 셀이 실행된다. 아래 방법대로 Mecab을 설치할 수 있다.


```python
! git clone https://github.com/SOMJANG/Mecab-ko-for-Google-Colab.git
```

    Cloning into 'Mecab-ko-for-Google-Colab'...
    remote: Enumerating objects: 115, done.[K
    remote: Counting objects: 100% (24/24), done.[K
    remote: Compressing objects: 100% (20/20), done.[K
    remote: Total 115 (delta 11), reused 10 (delta 3), pack-reused 91[K
    Receiving objects: 100% (115/115), 1.27 MiB | 6.19 MiB/s, done.
    Resolving deltas: 100% (50/50), done.
    


```python
cd Mecab-ko-for-Google-Colab
```

    /content/Mecab-ko-for-Google-Colab
    


```python
!bash install_mecab-ko_on_colab_light_220429.sh
```


```python
from konlpy.tag import Mecab
```

`name 'Tagger' is not defined` 오류가 뜨면 런타임 재실행!


```python
mecab = Mecab()
```

<br>

## 1. 영어 Word2Vec 만들기


```python
import re
import urllib.request
import zipfile
from lxml import etree
from nltk.tokenize import word_tokenize, sent_tokenize
```


```python
import nltk
nltk.download('punkt')
```

    [nltk_data] Downloading package punkt to /root/nltk_data...
    [nltk_data]   Unzipping tokenizers/punkt.zip.
    




    True




```python
# 훈련데이터 다운로드
urllib.request.urlretrieve("https://raw.githubusercontent.com/ukairia777/tensorflow-nlp-tutorial/main/09.%20Word%20Embedding/dataset/ted_en-20160408.xml", filename="ted_en-20160408.xml")
```




    ('ted_en-20160408.xml', <http.client.HTTPMessage at 0x7fe4c4e64b20>)




```python
targetXML = open('ted_en-20160408.xml', 'r', encoding='UTF8')
target_text = etree.parse(targetXML)

# xml 파일로부터 <content>와 </content> 사이의 내용만 가져온다.
parse_text = '\n'.join(target_text.xpath('//content/text()'))

# 정규 표현식의 sub 모듈을 통해 content 중간에 등장하는 (Audio), (Laughter) 등의 배경음 부분을 제거.
# 해당 코드는 괄호로 구성된 내용을 제거.
content_text = re.sub(r'\([^)]*\)', '', parse_text)

# 입력 코퍼스에 대해서 NLTK를 이용하여 문장 토큰화를 수행.
sent_text = sent_tokenize(content_text)

# 각 문장에 대해서 구두점을 제거하고, 대문자를 소문자로 변환.
normalized_text = []
for string in sent_text:
     tokens = re.sub(r"[^a-z0-9]+", " ", string.lower())
     normalized_text.append(tokens)

# 각 문장에 대해서 NLTK를 이용하여 단어 토큰화를 수행.
result = [word_tokenize(sentence) for sentence in normalized_text]
```


```python
print('총 샘플의 개수 : {}'.format(len(result)))
```

    총 샘플의 개수 : 273380
    


```python
# 샘플 3개만 출력
for line in result[:3]:
    print(line)
```

    ['here', 'are', 'two', 'reasons', 'companies', 'fail', 'they', 'only', 'do', 'more', 'of', 'the', 'same', 'or', 'they', 'only', 'do', 'what', 's', 'new']
    ['to', 'me', 'the', 'real', 'real', 'solution', 'to', 'quality', 'growth', 'is', 'figuring', 'out', 'the', 'balance', 'between', 'two', 'activities', 'exploration', 'and', 'exploitation']
    ['both', 'are', 'necessary', 'but', 'it', 'can', 'be', 'too', 'much', 'of', 'a', 'good', 'thing']
    


```python
from gensim.models import Word2Vec
from gensim.models import KeyedVectors
```

- `size` = 워드 벡터의 특징 값. 즉, 임베딩 된 벡터의 차원.
- `window` = 컨텍스트 윈도우 크기
- `min_count` = 단어 최소 빈도 수 제한 (빈도가 적은 단어들은 학습하지 않는다.)
- `workers` = 학습을 위한 프로세스 수
- `sg` = 0은 CBOW, 1은 Skip-gram.

CBOW는 target word 근처의 문맥을 파악하여 target word를 예측하는 방법이고, Skip-gram은 target word를 보고 문맥을 예측하는 방법이다. `window`란 인자는 근처 문맥의 단어를 몇개로 할지 그 크기를 설정하는 인자이다.

![neural language model vs word2vec](https://user-images.githubusercontent.com/115082062/213910013-2c91f210-090d-47f7-b842-33f64a3b2c50.png)



```python
model = Word2Vec(sentences=result, size=100, window=5, min_count=5, workers=4, sg=0)
```

`model.wv.most_similar()`를 통해 입력한 단어와 가장 유사한 단어를 출력할 수 있다. 코사인 유사도를 기반으로 출력해준다.


```python
model_result = model.wv.most_similar("ball")
print(model_result)
```

    [('button', 0.7781341075897217), ('rock', 0.7767028212547302), ('wheel', 0.7757996320724487), ('glass', 0.7734857201576233), ('rope', 0.7651770114898682), ('hole', 0.7528566718101501), ('balloon', 0.7417528033256531), ('grass', 0.7401809096336365), ('keyboard', 0.736878514289856), ('wire', 0.7237794399261475)]
    

유사도를 기반으로 산정된 벡터들이기 때문에 연산도 가능하다.


```python
model.wv.most_similar(positive=['woman'], negative=['man'])
```




    [('cancer', 0.3617965281009674),
     ('pregnant', 0.3389925956726074),
     ('failed', 0.3244003653526306),
     ('born', 0.322529673576355),
     ('married', 0.32099631428718567),
     ('her', 0.3199157416820526),
     ('older', 0.3169713616371155),
     ('child', 0.3151988387107849),
     ('united', 0.3039064109325409),
     ('patient', 0.29988113045692444)]




```python
model.wv.save_word2vec_format('eng_w2v') # 모델 저장
loaded_model = KeyedVectors.load_word2vec_format("eng_w2v") # 모델 로드
```


```python
model_result = loaded_model.most_similar("ball")
print(model_result)
```

    [('button', 0.7781341075897217), ('rock', 0.7767028212547302), ('wheel', 0.7757996320724487), ('glass', 0.7734857201576233), ('rope', 0.7651770114898682), ('hole', 0.7528566718101501), ('balloon', 0.7417528033256531), ('grass', 0.7401809096336365), ('keyboard', 0.736878514289856), ('wire', 0.7237794399261475)]
    

<br>

## 2. 한국어 Word2Vec 만들기


```python
import pandas as pd
import matplotlib.pyplot as plt
import urllib.request
from gensim.models.word2vec import Word2Vec
from konlpy.tag import Okt
import tqdm
```


```python
# 네이버 영화 리뷰 데이터 다운로드
urllib.request.urlretrieve("https://raw.githubusercontent.com/e9t/nsmc/master/ratings.txt", filename="ratings.txt")
```




    ('ratings.txt', <http.client.HTTPMessage at 0x7fb484cab0a0>)




```python
train_data = pd.read_table('ratings.txt')
```


```python
train_data.head()
```





  <div id="df-7475dda5-c0ea-4737-87d7-3f2b14fdd497">
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
      <th>id</th>
      <th>document</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8112052</td>
      <td>어릴때보고 지금다시봐도 재밌어요ㅋㅋ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8132799</td>
      <td>디자인을 배우는 학생으로, 외국디자이너와 그들이 일군 전통을 통해 발전해가는 문화산...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4655635</td>
      <td>폴리스스토리 시리즈는 1부터 뉴까지 버릴께 하나도 없음.. 최고.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9251303</td>
      <td>와.. 연기가 진짜 개쩔구나.. 지루할거라고 생각했는데 몰입해서 봤다.. 그래 이런...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10067386</td>
      <td>안개 자욱한 밤하늘에 떠 있는 초승달 같은 영화.</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-7475dda5-c0ea-4737-87d7-3f2b14fdd497')"
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
          document.querySelector('#df-7475dda5-c0ea-4737-87d7-3f2b14fdd497 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-7475dda5-c0ea-4737-87d7-3f2b14fdd497');
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
print(len(train_data))
```

    200000
    


```python
train_data.info() # 결측값이 존재하는 행 8개 있음
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 200000 entries, 0 to 199999
    Data columns (total 3 columns):
     #   Column    Non-Null Count   Dtype 
    ---  ------    --------------   ----- 
     0   id        200000 non-null  int64 
     1   document  199992 non-null  object
     2   label     200000 non-null  int64 
    dtypes: int64(2), object(1)
    memory usage: 4.6+ MB
    


```python
train_data = train_data.dropna(how = 'any') # 결측 값 존재하는 행 제거
```


```python
print(len(train_data)) # 8개 행이 사라짐
```

    199992
    


```python
# 정규 표현식을 통한 한글 외 문자 제거
train_data['document'] = train_data['document'].str.replace("[^ㄱ-ㅎㅏ-ㅣ가-힣 ]","")
```

    <ipython-input-8-d10eedfa8951>:2: FutureWarning: The default value of regex will change from True to False in a future version.
      train_data['document'] = train_data['document'].str.replace("[^ㄱ-ㅎㅏ-ㅣ가-힣 ]","")
    


```python
train_data.head()
```





  <div id="df-6ac77ccd-b9b4-41bb-9cef-8374a3476340">
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
      <th>id</th>
      <th>document</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8112052</td>
      <td>어릴때보고 지금다시봐도 재밌어요ㅋㅋ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8132799</td>
      <td>디자인을 배우는 학생으로 외국디자이너와 그들이 일군 전통을 통해 발전해가는 문화산업...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4655635</td>
      <td>폴리스스토리 시리즈는 부터 뉴까지 버릴께 하나도 없음 최고</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9251303</td>
      <td>와 연기가 진짜 개쩔구나 지루할거라고 생각했는데 몰입해서 봤다 그래 이런게 진짜 영화지</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10067386</td>
      <td>안개 자욱한 밤하늘에 떠 있는 초승달 같은 영화</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-6ac77ccd-b9b4-41bb-9cef-8374a3476340')"
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
          document.querySelector('#df-6ac77ccd-b9b4-41bb-9cef-8374a3476340 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-6ac77ccd-b9b4-41bb-9cef-8374a3476340');
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
from google import colab
colab.drive.mount("/content/drive")
```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    


```python
# 불용어 리스트 정의
with open('/content/drive/MyDrive/sentence/stopwords.txt',  encoding='cp949') as f:
    list_file = f.readlines()
    stopwords = list_file[0].split(",")
```

okt로 할 때 17분 걸리는 것이 mecab으로 할 땐 1분밖에 안 걸린다.


```python
okt = Okt()

tokenized_data = []
for sentence in tqdm.tqdm(train_data['document']):
    tokenized_sentence = okt.morphs(sentence, stem=True) # 토큰화
    stopwords_removed_sentence = [word for word in tokenized_sentence if not word in stopwords] # 불용어 제거
    tokenized_data.append(stopwords_removed_sentence)
```

    100%|██████████| 199992/199992 [17:30<00:00, 190.29it/s]
    


```python
tokenized_data = []
for sentence in tqdm.tqdm(train_data['document']):
    tokenized_sentence = mecab.morphs(sentence) # 토큰화
    stopwords_removed_sentence = [word for word in tokenized_sentence if not word in stopwords] # 불용어 제거
    tokenized_data.append(stopwords_removed_sentence)
```

    100%|██████████| 199992/199992 [01:18<00:00, 2549.23it/s]
    


```python
print(tokenized_data[:3])
```

    [['어릴', '보', '고', '봐도', '재밌', '어요'], ['디자인', '배우', '학생', '외국', '디자이너', '일군', '전통', '통해', '발전', '해', '문화', '산업', '부러웠', '는데', '사실', '나라', '어려운', '시절', '끝', '열정', '지킨', '노라노', '같', '전통', '있', '같', '사람', '꿈', '꾸', '고', '이뤄나갈', '수', '있', '다는', '감사', '합니다'], ['폴리스', '스토리', '시리즈', '뉴', '버릴', '께', '없', '음', '최고']]
    

불용어의 개수를 늘렸더니 예시보다 길이가 더 짧아졌다.


```python
# 리뷰 길이 분포 확인
print('리뷰의 최대 길이 :',max(len(review) for review in tokenized_data))
print('리뷰의 평균 길이 :',sum(map(len, tokenized_data))/len(tokenized_data))
plt.hist([len(review) for review in tokenized_data], bins=50)
plt.xlabel('length of samples')
plt.ylabel('number of samples')
plt.show()
```

    리뷰의 최대 길이 : 70
    리뷰의 평균 길이 : 10.689342573702948
    


    
<img src="https://user-images.githubusercontent.com/115082062/228714723-cfe6a40f-9be3-4180-9197-d711e3d25139.png">
    



```python
model = Word2Vec(sentences = tokenized_data, size = 100, window = 5, min_count = 5, workers = 4, sg = 0)
```


```python
model.wv.vectors.shape # 총 17806개의 단어가 100차원으로 구성되어있다.
```




    (17806, 100)




```python
print(model.wv.most_similar("이동진")) 
#같은 평론가인 김혜리, 박평식이 보인다. '식이'는 박평식의 별칭인 '평식이형'에서 떨어져 나온 것 같다.
```

    [('김혜리', 0.8537285327911377), ('식이', 0.8399664163589478), ('씨네', 0.8380143642425537), ('박평식', 0.8304151296615601), ('허지웅', 0.8267611861228943), ('순례', 0.8207776546478271), ('신분', 0.8180666565895081), ('충', 0.815009355545044), ('이용철', 0.811989426612854), ('성지', 0.810979962348938)]
    


```python
model.wv.most_similar(positive=['타짜'])
```




    [('공공', 0.9504997134208679),
     ('넘사벽', 0.9432716965675354),
     ('인정받다', 0.9430745840072632),
     ('세르', 0.9405409693717957),
     ('디스코', 0.9401938915252686),
     ('벤허', 0.9397081136703491),
     ('교본', 0.9386974573135376),
     ('의적', 0.9380705952644348),
     ('경지', 0.9379533529281616),
     ('애니메', 0.9374954700469971)]




```python
model.wv.most_similar(positive=['송강호'], negative=['주연'])
```




    [('휴일', 0.6941962242126465),
     ('엇음', 0.6847164034843445),
     ('퀵', 0.6747357249259949),
     ('고재', 0.669157862663269),
     ('서여', 0.6675720810890198),
     ('홀린', 0.6667022109031677),
     ('아보', 0.6652776598930359),
     ('원치않다', 0.6618590354919434),
     ('맞춤법', 0.661419153213501),
     ('녹차', 0.6606814861297607)]




```python
model.wv.similarity('송강호', '하정우')
```




    0.870262




```python
model.wv.similarity('송강호', '야구')
```




    0.19670776

<br>


## 3. 사전 훈련된 Word2Vec

구글에서 제공하는 사전 훈련된 Word2Vec 모델을 가져올 것이다. 이 모델은 3백만 개의 단어를 300차원의 임베딩 벡터로 만들어놓은 것이다.

예시 코드를 사용하다 `HTTP Error 404: Not Found` 에러가 나는 경우는

https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit

위 링크에서 모델을 다운받고 압축을 푼 후 구글 드라이브 적당한 디렉토리에 넣어두고 실행하면 된다.


```python
import gensim
import urllib.request

# 모델 저장해놓은 경로를 적어주면 된다
word2vec_model = gensim.models.KeyedVectors.load_word2vec_format('/content/drive/MyDrive/kubig/GoogleNews-vectors-negative300.bin', binary=True)
```

3백만 개의 단어가 저장된 모델이다보니 용량이 3기가가 넘는다.


```python
print(word2vec_model.vectors.shape)
```

    (3000000, 300)
    

`similarity()`를 통해 두 단어의 유사도를 출력할 수 있다.


```python
print(word2vec_model.similarity('baseball', 'football'))
print(word2vec_model.similarity('baseball', 'car'))
```

    0.6162001
    0.1090335
    
