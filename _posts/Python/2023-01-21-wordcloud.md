---
layout: single
title: "[Word Cloud & NLP] 텍스트를 전처리하고 워드클라우드로 나타내보기"
excerpt: "NLP에 쓰이는 각종 기법으로 텍스트를 전처리하고, 이를 통해 Wordcloud를 그려보자"

published: true

categories:
  - Python

tags: [python, Wordcloud, NLP]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2023-01-21 01:20:00
last_modified_at: 2023-01-21 01:20:00
---

<br>

예시 코드에 나온 워드 클라우드를 연습해보기 위해 새로운 데이터셋을 크롤링해보려 한다.

이번에 아파트 단지 내 헬스장에 새로 등록을 해서 프로틴 구매 욕구가 나날이 늘어나고 있던 참이라 마이프로틴의 제품 리뷰를 크롤링해서 워드 클라우드를 만들어보려 한다.

 1. 제품 리뷰를 크롤링한다
 2. 정규표현식으로 한글만 추린다
 3. 띄어쓰기를 교정한다(PyKospacing)
 4. 형태소로 토큰화한다(okt)
 5. 불용어를 제거한다
 6. 원하는 단어를 Konlpy 사전에 추가한다
 7. 워드클라우드를 그린다

 
 위같은 과정을 거쳐보겠다!

<br>

```python
!pip install selenium
```


```python
!pip install git+https://github.com/haven-jeon/PyKoSpacing.git
```

  

```python
import selenium
from selenium import webdriver
from selenium.webdriver import ActionChains

from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By

from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import Select
from selenium.webdriver.support.ui import WebDriverWait

from bs4 import BeautifulSoup

import pandas as pd
import numpy as np

import re

import pandas as pd
```


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    

<br>

## 1. 데이터 크롤링하기


```python
driver = webdriver.Chrome(executable_path = "C:/Users/희준/downloads/chromedriver_win32/chromedriver.exe")
```


```python
df = pd.DataFrame()

# 긁어올 클래스는 아래 두 가지(리뷰 제목, 리뷰 내용)
class_name = ['athenaProductReviews_reviewTitle', 'athenaProductReviews_reviewContent']

for i in range(100): # 1~100페이지를 순회하며 크롤링
    url = 'https://www.myprotein.co.kr/sports-nutrition/impact-whey-protein/10530943.reviews?pageNumber={}'.format(i+1)
    driver.get(url)
    driver.implicitly_wait(time_to_wait=5)
    soup = BeautifulSoup(driver.page_source, 'html')
    soup = soup.find(class_="athenaProductReviews_allReviewsContent") # 타겟이 될 class

    diction = {}
    for n in class_name:
        length = len(soup.find_all(class_=n)) # 한 페이지당 리뷰의 개수
        temp = [soup.find_all(class_=n)[k].text.strip() for k in range(0,length)]
        diction[n] = temp
        
    df_ = pd.DataFrame(diction)
    df = pd.concat([df, df_],axis=0)

df.reset_index(drop=True, inplace=True)
df.columns = ['title','content']
df['text'] = df['title']+df['content']
```

윗 부분까지는 vscode로 진행했다. 코랩에선 웹드라이버가 계속 오류가 나서 vscode로 크롤링을 하고 csv 파일로 만들어 코랩에서 이를 불러왔다.


```python
df = pd.read_csv("/content/drive/MyDrive/sentence/crawl_df.csv")
```


```python
df.head()
```





  <div id="df-6f6cd895-bd27-4617-b5f6-92a647e83798">
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
      <th>Unnamed: 0</th>
      <th>title</th>
      <th>content</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>처음 사먹어 보는데 맛있네요</td>
      <td>맛이 좋아서 꾸준히 먹고 있습니다.^^</td>
      <td>처음 사먹어 보는데 맛있네요맛이 좋아서 꾸준히 먹고 있습니다.^^</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>말해 뭐합니까 ㅋㅋㅋ 역시</td>
      <td>가성비 갑 제품 + 맛도 다양해서 ㅋㅋㅋ 항상 구매하고 있습니다요! \n\n    ...</td>
      <td>말해 뭐합니까 ㅋㅋㅋ 역시가성비 갑 제품 + 맛도 다양해서 ㅋㅋㅋ 항상 구매하고 있...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>좋넹요</td>
      <td>맛있습니다. 저번에 말차라떼도 먹어봤는데 그것보다 더 나은거 같네요</td>
      <td>좋넹요맛있습니다. 저번에 말차라떼도 먹어봤는데 그것보다 더 나은거 같네요</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>매우 만족합니다</td>
      <td>5키로는 정말 엄청크네요.\n    평점이 좋아서 스트로베리크림으로 주문했는데 좋습...</td>
      <td>매우 만족합니다5키로는 정말 엄청크네요.\n    평점이 좋아서 스트로베리크림으로 ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>마싯어요</td>
      <td>약간 달긴 하지만 물에 잘 녹고 가루날림이 거의없어 좋아요</td>
      <td>마싯어요약간 달긴 하지만 물에 잘 녹고 가루날림이 거의없어 좋아요</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-6f6cd895-bd27-4617-b5f6-92a647e83798')"
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
          document.querySelector('#df-6f6cd895-bd27-4617-b5f6-92a647e83798 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-6f6cd895-bd27-4617-b5f6-92a647e83798');
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

## 2. 정규표현식으로 한글 단어만 남기기

이제 정규표현식으로 한글 단어만 남기고 모두 제외한다.


```python
def extract_hangul(text):
  hangul = re.sub('[^가-힣]', ' ', text)
  return hangul
```


```python
example = extract_hangul(df['text'][1])
print("전처리 이전: ",df['text'][1])
print("전처리 이후: ",example)
```

    전처리 이전:  말해 뭐합니까 ㅋㅋㅋ 역시가성비 갑 제품 + 맛도 다양해서 ㅋㅋㅋ 항상 구매하고 있습니다요! 
    
        스트로베리 크림치즈가 가장 맛있었네요!
    전처리 이후:  말해 뭐합니까     역시가성비 갑 제품   맛도 다양해서     항상 구매하고 있습니다요        스트로베리 크림치즈가 가장 맛있었네요 
    

<br>

## 3. 띄어쓰기 교정(PyKoSpacing)

`PyKoSpacing`은 띄어쓰기가 되지 않은 문장을 띄어쓰기가 된 문장으로 교정해주는 패키지이다. 앞서 한글 외 다른 문자를 공백으로 치환했는데, 중첩된 공백들도 이 띄어쓰기 교정으로 하나의 공백으로 바꿔줄 수 있다.


```python
from pykospacing import Spacing
spacing = Spacing()
```


```python
def spacing_text(text):
  spaced_text = spacing(text)
  return spaced_text
```


```python
spaced = spacing_text(example)
print("전처리 이전: ",example)
print("전처리 이후: ",spaced)
```

    1/1 [==============================] - 1s 896ms/step
    전처리 이전:  말해 뭐합니까     역시가성비 갑 제품   맛도 다양해서     항상 구매하고 있습니다요        스트로베리 크림치즈가 가장 맛있었네요 
    전처리 이후:  말해 뭐 합니까 역시 가성비 갑 제품 맛도 다양해서 항상 구매하고 있습니다 요 스트로베리 크림치즈가 가장 맛있었네요
    

<br>

## 4. 형태소 분석기(okt)

한국어에서 토큰화를 해주는 대표적인 도구는 `konlpy`이다. 가장 대표적인 형태소 분석기는 `Okt` 모델이다.


```python
from konlpy.tag import Okt
okt = Okt()
```


```python
def extract_morphs(text):
  morphs_ = okt.morphs(text, stem=True)
  return morphs_
```


```python
morphs_ = extract_morphs(spaced)
print("전처리 이전: ",spaced)
print("전처리 이후: ",morphs_)
```

    전처리 이전:  말해 뭐 합니까 역시 가성비 갑 제품 맛도 다양해서 항상 구매하고 있습니다 요 스트로베리 크림치즈가 가장 맛있었네요
    전처리 이후:  ['말', '하다', '뭐', '합', '니까', '역시', '가성비', '갑', '제품', '맛', '도', '다양하다', '항상', '구매', '하고', '있다', '요', '스트로베리', '크림', '치즈', '가', '가장', '맛있다']
    

<br>

## 5. 불용어 제거

영어와 마찬가지로 한국어도 불용어를 지정해줘야 한다. '하다', '도', '는', '이다' 등 의미가 없는 어미나 조사같은 것을 지워주기로 하자.

불용어 참고 사이트: https://www.ranks.nl/stopwords/korean

위 불용어 자료에서 몇 개의 단어를 추가해서 드라이브에 저장해주었다.


```python
with open('/content/drive/MyDrive/sentence/stopwords.txt',  encoding='cp949') as f:
    list_file = f.readlines()

def remove_stopwords(text):
  stopwords = list_file[0].split(",")
  remove_stop = [x for x in text if x not in stopwords]
  return remove_stop
```


```python
remove_stop = remove_stopwords(morphs_)
print("전처리 이전: ",morphs_)
print("전처리 이후: ",remove_stop)
```

    전처리 이전:  ['말', '하다', '뭐', '합', '니까', '역시', '가성비', '갑', '제품', '맛', '도', '다양하다', '항상', '구매', '하고', '있다', '요', '스트로베리', '크림', '치즈', '가', '가장', '맛있다']
    전처리 이후:  ['말', '뭐', '합', '니까', '가성비', '갑', '제품', '맛', '다양하다', '항상', '구매', '스트로베리', '크림', '치즈', '맛있다']
    

추가적으로, 불용어로만 걸러내기엔 한계가 있고 아예 한 글자 단어들을 전부 없애주는 것이 낫다고 판단했다. 하지만, '맛', '향', '짱' 같은 단어들은 남겨두기로 했다.


```python
def remove_one(text):
  except_list = ['맛','향','짱']
  remove_one_ = [x for x in text if len(x)>1 or x in except_list]
  return remove_one_
```


```python
remove_one_ = remove_one(remove_stop)
print("전처리 이전: ",remove_stop)
print("전처리 이후: ",remove_one_)
```

    전처리 이전:  ['말', '뭐', '합', '니까', '가성비', '갑', '제품', '맛', '다양하다', '항상', '구매', '스트로베리', '크림', '치즈', '맛있다']
    전처리 이후:  ['니까', '가성비', '제품', '맛', '다양하다', '항상', '구매', '스트로베리', '크림', '치즈', '맛있다']
    

<br>

## 6. 원하는 단어 Konlpy 사전에 추가

그런데 몇 단어들을 찾아보니 `가성비`는 `가`, `성비`가 나뉘고 `프로틴`은 `프로`, `틴`이 나뉘는 안타까운 상황이 발생했다.

이런 경우, konlpy 사전에 직접 단어를 등재시키는 방법이 있다.

아래처럼 `os` 모듈을 사용하여 코랩 패키지에 저장된 konlpy 폴더에 손을 대는 것이다.

우선 `chdir`로 경로를 이동해주고 `makedirs`로 임시폴더를 만들어준다. 이 임시폴더에서 단어 사전을 수정한 뒤 원본 폴더에 저장해줄 것이다.


```python
import os

os.chdir('/usr/local/lib/python3.8/dist-packages/konlpy/java')
os.getcwd() 
os.makedirs('./aaaa')
```


```python
os.chdir('/usr/local/lib/python3.8/dist-packages/konlpy/java/aaaa') #임시 폴더로 이동
os.getcwd() 
```




    '/usr/local/lib/python3.8/dist-packages/konlpy/java/aaaa'



임시폴더에 konlpy 사전 파일의 압축을 풀어준다.


```python
!jar xvf ../open-korean-text-2.1.0.jar
```

      created: META-INF/
     inflated: META-INF/MANIFEST.MF
      created: org/
      created: org/openkoreantext/
      created: org/openkoreantext/processor/
      created: org/openkoreantext/processor/normalizer/
      created: org/openkoreantext/processor/phrase_extractor/
      created: org/openkoreantext/processor/qa/
      created: org/openkoreantext/processor/stemmer/
      created: org/openkoreantext/processor/tokenizer/
      created: org/openkoreantext/processor/tools/
      created: org/openkoreantext/processor/util/
      created: org/openkoreantext/processor/util/adjective/
      created: org/openkoreantext/processor/util/adverb/
      created: org/openkoreantext/processor/util/auxiliary/
      created: org/openkoreantext/processor/util/freq/
      created: org/openkoreantext/processor/util/josa/
      created: org/openkoreantext/processor/util/noun/
      created: org/openkoreantext/processor/util/substantives/
      created: org/openkoreantext/processor/util/typos/
      created: org/openkoreantext/processor/util/verb/
     inflated: org/openkoreantext/processor/KoreanPosJava.class
     inflated: org/openkoreantext/processor/KoreanTokenJava.class
     inflated: org/openkoreantext/processor/normalizer/KoreanNormalizer$.class
     inflated: org/openkoreantext/processor/normalizer/KoreanNormalizer$Segment$.class
     inflated: org/openkoreantext/processor/normalizer/KoreanNormalizer$Segment.class
     inflated: org/openkoreantext/processor/normalizer/KoreanNormalizer.class
     inflated: org/openkoreantext/processor/OpenKoreanTextProcessor$.class
     inflated: org/openkoreantext/processor/OpenKoreanTextProcessor.class
     inflated: org/openkoreantext/processor/OpenKoreanTextProcessorJava.class
     inflated: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$.class
     inflated: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$KoreanPhrase$.class
     inflated: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$KoreanPhrase.class
     inflated: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$PhraseBuffer$.class
     inflated: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$PhraseBuffer.class
     inflated: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor.class
     inflated: org/openkoreantext/processor/qa/BatchGetUnknownNouns$.class
     inflated: org/openkoreantext/processor/qa/BatchGetUnknownNouns$ChunkWithTweet$.class
     inflated: org/openkoreantext/processor/qa/BatchGetUnknownNouns$ChunkWithTweet.class
     inflated: org/openkoreantext/processor/qa/BatchGetUnknownNouns.class
     inflated: org/openkoreantext/processor/qa/BatchTokenizeTweets$.class
     inflated: org/openkoreantext/processor/qa/BatchTokenizeTweets$ParseTime$.class
     inflated: org/openkoreantext/processor/qa/BatchTokenizeTweets$ParseTime.class
     inflated: org/openkoreantext/processor/qa/BatchTokenizeTweets.class
     inflated: org/openkoreantext/processor/qa/KoreanProcessorSandbox$.class
     inflated: org/openkoreantext/processor/qa/KoreanProcessorSandbox.class
     inflated: org/openkoreantext/processor/stemmer/KoreanStemmer$.class
     inflated: org/openkoreantext/processor/stemmer/KoreanStemmer.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanChunk$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanChunk.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanChunker$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanChunker$ChunkMatch$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanChunker$ChunkMatch.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanChunker.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanDetokenizer$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanDetokenizer.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanSentenceSplitter$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanSentenceSplitter.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$CandidateParse$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$CandidateParse.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$KoreanToken$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$KoreanToken.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$PossibleTrie$.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer$PossibleTrie.class
     inflated: org/openkoreantext/processor/tokenizer/KoreanTokenizer.class
     inflated: org/openkoreantext/processor/tokenizer/ParsedChunk$.class
     inflated: org/openkoreantext/processor/tokenizer/ParsedChunk.class
     inflated: org/openkoreantext/processor/tokenizer/Sentence$.class
     inflated: org/openkoreantext/processor/tokenizer/Sentence.class
     inflated: org/openkoreantext/processor/tokenizer/TokenizerProfile$.class
     inflated: org/openkoreantext/processor/tokenizer/TokenizerProfile.class
     inflated: org/openkoreantext/processor/tools/CreateChunkParsingCandidates$.class
     inflated: org/openkoreantext/processor/tools/CreateChunkParsingCandidates.class
     inflated: org/openkoreantext/processor/tools/CreateConjugationExamples$.class
     inflated: org/openkoreantext/processor/tools/CreateConjugationExamples$ConjugationExample$.class
     inflated: org/openkoreantext/processor/tools/CreateConjugationExamples$ConjugationExample.class
     inflated: org/openkoreantext/processor/tools/CreateConjugationExamples.class
     inflated: org/openkoreantext/processor/tools/CreateParsingExamples$.class
     inflated: org/openkoreantext/processor/tools/CreateParsingExamples$ParsingExample$.class
     inflated: org/openkoreantext/processor/tools/CreateParsingExamples$ParsingExample.class
     inflated: org/openkoreantext/processor/tools/CreateParsingExamples.class
     inflated: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples$.class
     inflated: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples$PhraseExample$.class
     inflated: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples$PhraseExample.class
     inflated: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples.class
     inflated: org/openkoreantext/processor/tools/DeduplicateAndSortDictionaries$.class
     inflated: org/openkoreantext/processor/tools/DeduplicateAndSortDictionaries.class
     inflated: org/openkoreantext/processor/tools/Runnable.class
     inflated: org/openkoreantext/processor/tools/UpdateAllTheExamples$.class
     inflated: org/openkoreantext/processor/tools/UpdateAllTheExamples.class
     inflated: org/openkoreantext/processor/util/adjective/adjective.txt
     inflated: org/openkoreantext/processor/util/adverb/adverb.txt
     inflated: org/openkoreantext/processor/util/auxiliary/conjunctions.txt
     inflated: org/openkoreantext/processor/util/auxiliary/determiner.txt
     inflated: org/openkoreantext/processor/util/auxiliary/exclamation.txt
     inflated: org/openkoreantext/processor/util/CharacterUtils$CharacterBuffer.class
     inflated: org/openkoreantext/processor/util/CharacterUtils$Java4CharacterUtils.class
     inflated: org/openkoreantext/processor/util/CharacterUtils$Java5CharacterUtils.class
     inflated: org/openkoreantext/processor/util/CharacterUtils.class
     inflated: org/openkoreantext/processor/util/CharArrayMap$1.class
     inflated: org/openkoreantext/processor/util/CharArrayMap$EmptyCharArrayMap.class
     inflated: org/openkoreantext/processor/util/CharArrayMap$EntryIterator.class
     inflated: org/openkoreantext/processor/util/CharArrayMap$EntrySet.class
     inflated: org/openkoreantext/processor/util/CharArrayMap$MapEntry.class
     inflated: org/openkoreantext/processor/util/CharArrayMap$UnmodifiableCharArrayMap.class
     inflated: org/openkoreantext/processor/util/CharArrayMap.class
     inflated: org/openkoreantext/processor/util/CharArraySet.class
     inflated: org/openkoreantext/processor/util/example_chunks.txt
     inflated: org/openkoreantext/processor/util/example_tweets.txt
     inflated: org/openkoreantext/processor/util/freq/entity-freq.txt.gz
     inflated: org/openkoreantext/processor/util/Hangul$.class
     inflated: org/openkoreantext/processor/util/Hangul$DoubleCoda$.class
     inflated: org/openkoreantext/processor/util/Hangul$DoubleCoda.class
     inflated: org/openkoreantext/processor/util/Hangul$HangulChar$.class
     inflated: org/openkoreantext/processor/util/Hangul$HangulChar.class
     inflated: org/openkoreantext/processor/util/Hangul.class
     inflated: org/openkoreantext/processor/util/josa/josa.txt
     inflated: org/openkoreantext/processor/util/KoreanConjugation$.class
     inflated: org/openkoreantext/processor/util/KoreanConjugation.class
     inflated: org/openkoreantext/processor/util/KoreanDictionaryProvider$.class
     inflated: org/openkoreantext/processor/util/KoreanDictionaryProvider.class
     inflated: org/openkoreantext/processor/util/KoreanPos$.class
     inflated: org/openkoreantext/processor/util/KoreanPos$KoreanPosTrie$.class
     inflated: org/openkoreantext/processor/util/KoreanPos$KoreanPosTrie.class
     inflated: org/openkoreantext/processor/util/KoreanPos.class
     inflated: org/openkoreantext/processor/util/KoreanSubstantive$.class
     inflated: org/openkoreantext/processor/util/KoreanSubstantive.class
     inflated: org/openkoreantext/processor/util/noun/bible.txt
     inflated: org/openkoreantext/processor/util/noun/company_names.txt
     inflated: org/openkoreantext/processor/util/noun/congress.txt
     inflated: org/openkoreantext/processor/util/noun/entities.txt
     inflated: org/openkoreantext/processor/util/noun/foreign.txt
     inflated: org/openkoreantext/processor/util/noun/geolocations.txt
     inflated: org/openkoreantext/processor/util/noun/kpop.txt
     inflated: org/openkoreantext/processor/util/noun/lol.txt
     inflated: org/openkoreantext/processor/util/noun/names.txt
     inflated: org/openkoreantext/processor/util/noun/nouns.txt
     inflated: org/openkoreantext/processor/util/noun/pokemon.txt
     inflated: org/openkoreantext/processor/util/noun/profane.txt
     inflated: org/openkoreantext/processor/util/noun/slangs.txt
     inflated: org/openkoreantext/processor/util/noun/spam.txt
     inflated: org/openkoreantext/processor/util/noun/twitter.txt
     inflated: org/openkoreantext/processor/util/noun/wikipedia_title_nouns.txt
     inflated: org/openkoreantext/processor/util/substantives/family_names.txt
     inflated: org/openkoreantext/processor/util/substantives/given_names.txt
     inflated: org/openkoreantext/processor/util/substantives/modifier.txt
     inflated: org/openkoreantext/processor/util/substantives/suffix.txt
     inflated: org/openkoreantext/processor/util/typos/typos.txt
     inflated: org/openkoreantext/processor/util/verb/eomi.txt
     inflated: org/openkoreantext/processor/util/verb/pre_eomi.txt
     inflated: org/openkoreantext/processor/util/verb/verb.txt
     inflated: org/openkoreantext/processor/util/verb/verb_prefix.txt
      created: META-INF/maven/
      created: META-INF/maven/org.openkoreantext/
      created: META-INF/maven/org.openkoreantext/open-korean-text/
     inflated: META-INF/maven/org.openkoreantext/open-korean-text/pom.xml
     inflated: META-INF/maven/org.openkoreantext/open-korean-text/pom.properties
    

압축이 잘 풀렸으니 이제 명사들이 등재되어 있는 names.txt를 열어보도록 하자.


```python
with open(f"/usr/local/lib/python3.8/dist-packages/konlpy/java/aaaa/org/openkoreantext/processor/util/noun/names.txt") as f:
    data = f.read()
```

아래처럼 다양한 단어들이 저장되어있는 것을 볼 수 있다. 여러 사용자들의 니즈를 반영하여 몇 단어들이 추가된 것으로 보인다.


```python
data
```




    '가몽\n가온\n갓세븐\n강새이\n게임닉가\n관우\n귀여미\n규\n김유이\n김준면\n까까런\n노컷\n누너예\n니노\n다마고치\n다이무스\n대학생\n데이브\n도요토미\n동운\n동이\n두주니\n디시인사이드\n디오\n라몹\n라스\n라옵\n멍구\n메이든\n명덕\n명량\n문민정부\n미네\n방엘리\n병헌\n붓다\n비정상회담\n빼빼로\n삼풍\n샤인온미\n성식\n성열\n세라문\n세라복\n세종대왕\n손권\n손책\n쇼미더머니\n쇼챔\n순규\n스라소니\n신동아\n신쓰패밀리\n신아라\n아베\n안상홍\n안홍준\n여누\n여랑\n여포\n연합\n오꾸닭\n요섭\n웃찾사\n원식\n유병언\n유비\n유이\n윤기형\n이나단\n이명박\n이완용\n임창용\n자괴\n자니윤\n자대련\n자유\n재중이\n전교조\n정윤회\n제갈량\n조자룡\n조조\n준면\n지오디\n지존파\n진영오\n차작가\n차트\n창섭\n챠트\n첸\n코르사주\n하무열\n하용파쿠\n혁재\n현이\n현태\n혜미\n'



일단 세 개의 단어를 추가해보도록 하자. 그리고 쓰기 모드로 변경하여 새롭게 파일을 저장한다.


```python
data += '프로틴\n가성비\n밀크티\n'

with open("/usr/local/lib/python3.8/dist-packages/konlpy/java/aaaa/org/openkoreantext/processor/util/noun/names.txt", 'w') as f:
    f.write(data)
```

이제 다시 파일을 압축시키면 완료! 런타임을 재실행해야 제대로 반영이 된다.


```python
!jar cvf ../open-korean-text-2.1.0.jar * 
```

    added manifest
    ignoring entry META-INF/
    adding: META-INF/maven/(in = 0) (out= 0)(stored 0%)
    adding: META-INF/maven/org.openkoreantext/(in = 0) (out= 0)(stored 0%)
    adding: META-INF/maven/org.openkoreantext/open-korean-text/(in = 0) (out= 0)(stored 0%)
    adding: META-INF/maven/org.openkoreantext/open-korean-text/pom.xml(in = 9127) (out= 2208)(deflated 75%)
    adding: META-INF/maven/org.openkoreantext/open-korean-text/pom.properties(in = 119) (out= 110)(deflated 7%)
    ignoring entry META-INF/MANIFEST.MF
    adding: org/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/phrase_extractor/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$.class(in = 37855) (out= 12404)(deflated 67%)
    adding: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$KoreanPhrase$.class(in = 3358) (out= 1148)(deflated 65%)
    adding: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$KoreanPhrase.class(in = 6942) (out= 2591)(deflated 62%)
    adding: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$PhraseBuffer$.class(in = 3564) (out= 1087)(deflated 69%)
    adding: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor.class(in = 6005) (out= 3967)(deflated 33%)
    adding: org/openkoreantext/processor/phrase_extractor/KoreanPhraseExtractor$PhraseBuffer.class(in = 4857) (out= 1609)(deflated 66%)
    adding: org/openkoreantext/processor/OpenKoreanTextProcessor.class(in = 4428) (out= 1997)(deflated 54%)
    adding: org/openkoreantext/processor/tools/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/tools/CreateParsingExamples$ParsingExample.class(in = 3718) (out= 1429)(deflated 61%)
    adding: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples$PhraseExample.class(in = 3825) (out= 1443)(deflated 62%)
    adding: org/openkoreantext/processor/tools/UpdateAllTheExamples$.class(in = 4383) (out= 1794)(deflated 59%)
    adding: org/openkoreantext/processor/tools/CreateParsingExamples$.class(in = 5517) (out= 2327)(deflated 57%)
    adding: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples.class(in = 2516) (out= 1784)(deflated 29%)
    adding: org/openkoreantext/processor/tools/CreateConjugationExamples.class(in = 2367) (out= 1659)(deflated 29%)
    adding: org/openkoreantext/processor/tools/DeduplicateAndSortDictionaries$.class(in = 7154) (out= 3133)(deflated 56%)
    adding: org/openkoreantext/processor/tools/CreateChunkParsingCandidates.class(in = 833) (out= 566)(deflated 32%)
    adding: org/openkoreantext/processor/tools/CreateConjugationExamples$ConjugationExample.class(in = 3569) (out= 1415)(deflated 60%)
    adding: org/openkoreantext/processor/tools/UpdateAllTheExamples.class(in = 1089) (out= 756)(deflated 30%)
    adding: org/openkoreantext/processor/tools/DeduplicateAndSortDictionaries.class(in = 1117) (out= 823)(deflated 26%)
    adding: org/openkoreantext/processor/tools/CreateConjugationExamples$ConjugationExample$.class(in = 2787) (out= 997)(deflated 64%)
    adding: org/openkoreantext/processor/tools/Runnable.class(in = 964) (out= 694)(deflated 28%)
    adding: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples$.class(in = 5989) (out= 2472)(deflated 58%)
    adding: org/openkoreantext/processor/tools/CreateChunkParsingCandidates$.class(in = 5487) (out= 2305)(deflated 57%)
    adding: org/openkoreantext/processor/tools/CreateConjugationExamples$.class(in = 5881) (out= 2385)(deflated 59%)
    adding: org/openkoreantext/processor/tools/CreatePhraseExtractionExamples$PhraseExample$.class(in = 2974) (out= 1020)(deflated 65%)
    adding: org/openkoreantext/processor/tools/CreateParsingExamples$ParsingExample$.class(in = 2842) (out= 1010)(deflated 64%)
    adding: org/openkoreantext/processor/tools/CreateParsingExamples.class(in = 2427) (out= 1732)(deflated 28%)
    adding: org/openkoreantext/processor/util/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/CharArrayMap$EntryIterator.class(in = 3118) (out= 1270)(deflated 59%)
    adding: org/openkoreantext/processor/util/CharArrayMap$EntrySet.class(in = 2788) (out= 1181)(deflated 57%)
    adding: org/openkoreantext/processor/util/Hangul$HangulChar$.class(in = 2389) (out= 1022)(deflated 57%)
    adding: org/openkoreantext/processor/util/CharArrayMap$UnmodifiableCharArrayMap.class(in = 3062) (out= 859)(deflated 71%)
    adding: org/openkoreantext/processor/util/KoreanDictionaryProvider$.class(in = 20197) (out= 7488)(deflated 62%)
    adding: org/openkoreantext/processor/util/noun/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/noun/bible.txt(in = 2032) (out= 1005)(deflated 50%)
    adding: org/openkoreantext/processor/util/noun/twitter.txt(in = 254) (out= 162)(deflated 36%)
    adding: org/openkoreantext/processor/util/noun/congress.txt(in = 26494) (out= 9351)(deflated 64%)
    adding: org/openkoreantext/processor/util/noun/pokemon.txt(in = 1781) (out= 965)(deflated 45%)
    adding: org/openkoreantext/processor/util/noun/nouns.txt(in = 219319) (out= 84134)(deflated 61%)
    adding: org/openkoreantext/processor/util/noun/lol.txt(in = 1337) (out= 744)(deflated 44%)
    adding: org/openkoreantext/processor/util/noun/entities.txt(in = 108142) (out= 44095)(deflated 59%)
    adding: org/openkoreantext/processor/util/noun/foreign.txt(in = 10485) (out= 4686)(deflated 55%)
    adding: org/openkoreantext/processor/util/noun/slangs.txt(in = 1565) (out= 921)(deflated 41%)
    adding: org/openkoreantext/processor/util/noun/names.txt(in = 933) (out= 567)(deflated 39%)
    adding: org/openkoreantext/processor/util/noun/spam.txt(in = 202) (out= 157)(deflated 22%)
    adding: org/openkoreantext/processor/util/noun/kpop.txt(in = 5371) (out= 2495)(deflated 53%)
    adding: org/openkoreantext/processor/util/noun/wikipedia_title_nouns.txt(in = 1971964) (out= 686708)(deflated 65%)
    adding: org/openkoreantext/processor/util/noun/geolocations.txt(in = 5423) (out= 2267)(deflated 58%)
    adding: org/openkoreantext/processor/util/noun/company_names.txt(in = 1631) (out= 905)(deflated 44%)
    adding: org/openkoreantext/processor/util/noun/profane.txt(in = 569) (out= 337)(deflated 40%)
    adding: org/openkoreantext/processor/util/Hangul$HangulChar.class(in = 3185) (out= 1421)(deflated 55%)
    adding: org/openkoreantext/processor/util/KoreanConjugation$.class(in = 22440) (out= 7972)(deflated 64%)
    adding: org/openkoreantext/processor/util/CharArrayMap$EmptyCharArrayMap.class(in = 2053) (out= 725)(deflated 64%)
    adding: org/openkoreantext/processor/util/KoreanPos$KoreanPosTrie$.class(in = 3157) (out= 1089)(deflated 65%)
    adding: org/openkoreantext/processor/util/CharacterUtils$Java5CharacterUtils.class(in = 2824) (out= 1393)(deflated 50%)
    adding: org/openkoreantext/processor/util/adverb/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/adverb/adverb.txt(in = 65572) (out= 18741)(deflated 71%)
    adding: org/openkoreantext/processor/util/CharacterUtils$CharacterBuffer.class(in = 1634) (out= 659)(deflated 59%)
    adding: org/openkoreantext/processor/util/typos/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/typos/typos.txt(in = 5146) (out= 2300)(deflated 55%)
    adding: org/openkoreantext/processor/util/CharArrayMap$1.class(in = 1531) (out= 571)(deflated 62%)
    adding: org/openkoreantext/processor/util/KoreanDictionaryProvider.class(in = 3430) (out= 2003)(deflated 41%)
    adding: org/openkoreantext/processor/util/CharArrayMap$MapEntry.class(in = 2433) (out= 1078)(deflated 55%)
    adding: org/openkoreantext/processor/util/CharArrayMap.class(in = 11923) (out= 4878)(deflated 59%)
    adding: org/openkoreantext/processor/util/Hangul$DoubleCoda.class(in = 2930) (out= 1348)(deflated 53%)
    adding: org/openkoreantext/processor/util/KoreanPos.class(in = 6890) (out= 3877)(deflated 43%)
    adding: org/openkoreantext/processor/util/Hangul$.class(in = 7625) (out= 3114)(deflated 59%)
    adding: org/openkoreantext/processor/util/example_chunks.txt(in = 522978) (out= 163253)(deflated 68%)
    adding: org/openkoreantext/processor/util/adjective/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/adjective/adjective.txt(in = 27250) (out= 9357)(deflated 65%)
    adding: org/openkoreantext/processor/util/Hangul$DoubleCoda$.class(in = 2155) (out= 963)(deflated 55%)
    adding: org/openkoreantext/processor/util/KoreanPos$.class(in = 12173) (out= 4669)(deflated 61%)
    adding: org/openkoreantext/processor/util/CharacterUtils$Java4CharacterUtils.class(in = 2577) (out= 1282)(deflated 50%)
    adding: org/openkoreantext/processor/util/CharacterUtils.class(in = 3703) (out= 1716)(deflated 53%)
    adding: org/openkoreantext/processor/util/josa/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/josa/josa.txt(in = 5219) (out= 1476)(deflated 71%)
    adding: org/openkoreantext/processor/util/KoreanSubstantive.class(in = 1336) (out= 1087)(deflated 18%)
    adding: org/openkoreantext/processor/util/CharArraySet.class(in = 4886) (out= 1944)(deflated 60%)
    adding: org/openkoreantext/processor/util/KoreanSubstantive$.class(in = 14990) (out= 6195)(deflated 58%)
    adding: org/openkoreantext/processor/util/KoreanConjugation.class(in = 1550) (out= 1237)(deflated 20%)
    adding: org/openkoreantext/processor/util/example_tweets.txt(in = 86271) (out= 38336)(deflated 55%)
    adding: org/openkoreantext/processor/util/substantives/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/substantives/family_names.txt(in = 299) (out= 197)(deflated 34%)
    adding: org/openkoreantext/processor/util/substantives/modifier.txt(in = 3395) (out= 1346)(deflated 60%)
    adding: org/openkoreantext/processor/util/substantives/given_names.txt(in = 1407) (out= 609)(deflated 56%)
    adding: org/openkoreantext/processor/util/substantives/suffix.txt(in = 325) (out= 224)(deflated 31%)
    adding: org/openkoreantext/processor/util/verb/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/verb/verb_prefix.txt(in = 47) (out= 52)(deflated -10%)
    adding: org/openkoreantext/processor/util/verb/pre_eomi.txt(in = 285) (out= 185)(deflated 35%)
    adding: org/openkoreantext/processor/util/verb/eomi.txt(in = 12618) (out= 3556)(deflated 71%)
    adding: org/openkoreantext/processor/util/verb/verb.txt(in = 32758) (out= 10723)(deflated 67%)
    adding: org/openkoreantext/processor/util/auxiliary/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/auxiliary/conjunctions.txt(in = 398) (out= 189)(deflated 52%)
    adding: org/openkoreantext/processor/util/auxiliary/exclamation.txt(in = 1759) (out= 757)(deflated 56%)
    adding: org/openkoreantext/processor/util/auxiliary/determiner.txt(in = 63) (out= 63)(deflated 0%)
    adding: org/openkoreantext/processor/util/KoreanPos$KoreanPosTrie.class(in = 4390) (out= 1610)(deflated 63%)
    adding: org/openkoreantext/processor/util/freq/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/util/freq/entity-freq.txt.gz(in = 385753) (out= 385873)(deflated 0%)
    adding: org/openkoreantext/processor/util/Hangul.class(in = 3981) (out= 2731)(deflated 31%)
    adding: org/openkoreantext/processor/OpenKoreanTextProcessor$.class(in = 7716) (out= 2400)(deflated 68%)
    adding: org/openkoreantext/processor/OpenKoreanTextProcessorJava.class(in = 5901) (out= 1995)(deflated 66%)
    adding: org/openkoreantext/processor/normalizer/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/normalizer/KoreanNormalizer$.class(in = 11553) (out= 5102)(deflated 55%)
    adding: org/openkoreantext/processor/normalizer/KoreanNormalizer$Segment.class(in = 3470) (out= 1422)(deflated 59%)
    adding: org/openkoreantext/processor/normalizer/KoreanNormalizer$Segment$.class(in = 2577) (out= 989)(deflated 61%)
    adding: org/openkoreantext/processor/normalizer/KoreanNormalizer.class(in = 2980) (out= 2181)(deflated 26%)
    adding: org/openkoreantext/processor/KoreanTokenJava.class(in = 1976) (out= 931)(deflated 52%)
    adding: org/openkoreantext/processor/tokenizer/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$KoreanToken.class(in = 6336) (out= 2633)(deflated 58%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$.class(in = 25564) (out= 8451)(deflated 66%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$KoreanToken$.class(in = 3976) (out= 1402)(deflated 64%)
    adding: org/openkoreantext/processor/tokenizer/KoreanDetokenizer.class(in = 1660) (out= 1133)(deflated 31%)
    adding: org/openkoreantext/processor/tokenizer/TokenizerProfile.class(in = 16403) (out= 6666)(deflated 59%)
    adding: org/openkoreantext/processor/tokenizer/Sentence$.class(in = 2339) (out= 1012)(deflated 56%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$CandidateParse$.class(in = 3459) (out= 1111)(deflated 67%)
    adding: org/openkoreantext/processor/tokenizer/TokenizerProfile$.class(in = 8169) (out= 2408)(deflated 70%)
    adding: org/openkoreantext/processor/tokenizer/KoreanSentenceSplitter$.class(in = 3359) (out= 1497)(deflated 55%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$PossibleTrie$.class(in = 2862) (out= 1067)(deflated 62%)
    adding: org/openkoreantext/processor/tokenizer/KoreanDetokenizer$.class(in = 9889) (out= 4174)(deflated 57%)
    adding: org/openkoreantext/processor/tokenizer/KoreanChunker$ChunkMatch$.class(in = 2893) (out= 1136)(deflated 60%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$CandidateParse.class(in = 4616) (out= 1645)(deflated 64%)
    adding: org/openkoreantext/processor/tokenizer/KoreanChunker$.class(in = 17975) (out= 6790)(deflated 62%)
    adding: org/openkoreantext/processor/tokenizer/Sentence.class(in = 5985) (out= 3125)(deflated 47%)
    adding: org/openkoreantext/processor/tokenizer/ParsedChunk$.class(in = 3973) (out= 1500)(deflated 62%)
    adding: org/openkoreantext/processor/tokenizer/KoreanChunker$ChunkMatch.class(in = 4043) (out= 1704)(deflated 57%)
    adding: org/openkoreantext/processor/tokenizer/KoreanChunk.class(in = 5611) (out= 2907)(deflated 48%)
    adding: org/openkoreantext/processor/tokenizer/ParsedChunk.class(in = 20517) (out= 8914)(deflated 56%)
    adding: org/openkoreantext/processor/tokenizer/KoreanSentenceSplitter.class(in = 1092) (out= 712)(deflated 34%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer.class(in = 6968) (out= 4384)(deflated 37%)
    adding: org/openkoreantext/processor/tokenizer/KoreanTokenizer$PossibleTrie.class(in = 3399) (out= 1429)(deflated 57%)
    adding: org/openkoreantext/processor/tokenizer/KoreanChunk$.class(in = 2361) (out= 1010)(deflated 57%)
    adding: org/openkoreantext/processor/tokenizer/KoreanChunker.class(in = 4232) (out= 2887)(deflated 31%)
    adding: org/openkoreantext/processor/qa/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/qa/BatchTokenizeTweets$.class(in = 10867) (out= 4401)(deflated 59%)
    adding: org/openkoreantext/processor/qa/BatchTokenizeTweets.class(in = 2720) (out= 2031)(deflated 25%)
    adding: org/openkoreantext/processor/qa/BatchGetUnknownNouns$ChunkWithTweet.class(in = 3080) (out= 1338)(deflated 56%)
    adding: org/openkoreantext/processor/qa/BatchTokenizeTweets$ParseTime.class(in = 3153) (out= 1446)(deflated 54%)
    adding: org/openkoreantext/processor/qa/BatchGetUnknownNouns$.class(in = 8751) (out= 3532)(deflated 59%)
    adding: org/openkoreantext/processor/qa/KoreanProcessorSandbox$.class(in = 1464) (out= 823)(deflated 43%)
    adding: org/openkoreantext/processor/qa/KoreanProcessorSandbox.class(in = 824) (out= 575)(deflated 30%)
    adding: org/openkoreantext/processor/qa/BatchTokenizeTweets$ParseTime$.class(in = 2418) (out= 1015)(deflated 58%)
    adding: org/openkoreantext/processor/qa/BatchGetUnknownNouns$ChunkWithTweet$.class(in = 2376) (out= 952)(deflated 59%)
    adding: org/openkoreantext/processor/qa/BatchGetUnknownNouns.class(in = 2307) (out= 1702)(deflated 26%)
    adding: org/openkoreantext/processor/KoreanPosJava.class(in = 2457) (out= 1241)(deflated 49%)
    adding: org/openkoreantext/processor/stemmer/(in = 0) (out= 0)(stored 0%)
    adding: org/openkoreantext/processor/stemmer/KoreanStemmer.class(in = 1439) (out= 972)(deflated 32%)
    adding: org/openkoreantext/processor/stemmer/KoreanStemmer$.class(in = 7044) (out= 2892)(deflated 58%)
    

원래 두 단어로 나뉘어서 나오던 것들이 올바르게 한 단어로 출력되는 것을 볼 수 있다!


```python
print(okt.nouns("가성비"))
print(okt.nouns("프로틴"))
```

    ['가성비']
    ['프로틴']
    

<br>

전 과정을 하나로 통합하면 아래와 같다.


```python
from konlpy.tag import Okt
okt = Okt()

from pykospacing import Spacing
spacing = Spacing()

except_list = ['맛','향','짱']

with open('/content/drive/MyDrive/sentence/stopwords.txt',  encoding='cp949') as f:
    list_file = f.readlines()
    stopwords = list_file[0].split(",")

def review_to_words(raw_review):
  text = re.sub('[^가-힣]', ' ', raw_review)
  text = spacing(text)
  text = okt.morphs(text, stem=True)
  text = [x for x in text if x not in stopwords]
  text = [x for x in text if len(x)>1 or x in except_list]
  text = " ".join(text)
  return text
```

<br>

이제 `words_list`를 만들어 전처리한 단어들을 하나의 리스트에 모아주자. 띄어쓰기 교정 과정에서 약간 지연되는 것 같다.



```python
import tqdm
df_len = df.shape[0]
words_list = []
for i in range(df_len):
  words_list.append(review_to_words(df['text'][i]))
```

    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 51ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 63ms/step
    1/1 [==============================] - 0s 54ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 51ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 49ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 53ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 55ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 53ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 55ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 53ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 49ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 32ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 55ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 52ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 49ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 50ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 49ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 69ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 50ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 70ms/step
    1/1 [==============================] - 0s 59ms/step
    1/1 [==============================] - 0s 70ms/step
    1/1 [==============================] - 0s 75ms/step
    1/1 [==============================] - 0s 51ms/step
    1/1 [==============================] - 0s 58ms/step
    1/1 [==============================] - 0s 58ms/step
    1/1 [==============================] - 0s 60ms/step
    1/1 [==============================] - 0s 59ms/step
    1/1 [==============================] - 0s 75ms/step
    1/1 [==============================] - 0s 81ms/step
    1/1 [==============================] - 0s 63ms/step
    1/1 [==============================] - 0s 64ms/step
    1/1 [==============================] - 0s 68ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 63ms/step
    1/1 [==============================] - 0s 71ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 76ms/step
    1/1 [==============================] - 0s 63ms/step
    1/1 [==============================] - 0s 73ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 57ms/step
    1/1 [==============================] - 0s 64ms/step
    1/1 [==============================] - 0s 78ms/step
    1/1 [==============================] - 0s 69ms/step
    1/1 [==============================] - 0s 73ms/step
    1/1 [==============================] - 0s 78ms/step
    1/1 [==============================] - 0s 58ms/step
    1/1 [==============================] - 0s 65ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 73ms/step
    1/1 [==============================] - 0s 57ms/step
    1/1 [==============================] - 0s 55ms/step
    1/1 [==============================] - 0s 50ms/step
    1/1 [==============================] - 0s 73ms/step
    1/1 [==============================] - 0s 53ms/step
    1/1 [==============================] - 0s 73ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 66ms/step
    1/1 [==============================] - 0s 69ms/step
    1/1 [==============================] - 0s 82ms/step
    1/1 [==============================] - 0s 60ms/step
    1/1 [==============================] - 0s 85ms/step
    1/1 [==============================] - 0s 67ms/step
    1/1 [==============================] - 0s 69ms/step
    1/1 [==============================] - 0s 72ms/step
    1/1 [==============================] - 0s 108ms/step
    1/1 [==============================] - 0s 121ms/step
    1/1 [==============================] - 0s 67ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 70ms/step
    1/1 [==============================] - 0s 91ms/step
    1/1 [==============================] - 0s 65ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 77ms/step
    1/1 [==============================] - 0s 58ms/step
    1/1 [==============================] - 0s 56ms/step
    1/1 [==============================] - 0s 68ms/step
    1/1 [==============================] - 0s 64ms/step
    1/1 [==============================] - 0s 67ms/step
    1/1 [==============================] - 0s 60ms/step
    1/1 [==============================] - 0s 55ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 59ms/step
    1/1 [==============================] - 0s 59ms/step
    1/1 [==============================] - 0s 66ms/step
    1/1 [==============================] - 0s 56ms/step
    1/1 [==============================] - 0s 68ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 61ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 75ms/step
    1/1 [==============================] - 0s 62ms/step
    1/1 [==============================] - 0s 65ms/step
    1/1 [==============================] - 0s 70ms/step
    1/1 [==============================] - 0s 66ms/step
    1/1 [==============================] - 0s 68ms/step
    1/1 [==============================] - 0s 57ms/step
    1/1 [==============================] - 0s 67ms/step
    1/1 [==============================] - 0s 57ms/step
    1/1 [==============================] - 0s 67ms/step
    1/1 [==============================] - 0s 56ms/step
    1/1 [==============================] - 0s 63ms/step
    1/1 [==============================] - 0s 70ms/step
    1/1 [==============================] - 0s 65ms/step
    1/1 [==============================] - 0s 54ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 50ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 53ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 44ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 49ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 46ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 64ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 54ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 40ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 43ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 50ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 33ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 42ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 48ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 34ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 35ms/step
    1/1 [==============================] - 0s 45ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 47ms/step
    1/1 [==============================] - 0s 39ms/step
    1/1 [==============================] - 0s 38ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 41ms/step
    1/1 [==============================] - 0s 49ms/step
    1/1 [==============================] - 0s 37ms/step
    1/1 [==============================] - 0s 36ms/step
    1/1 [==============================] - 0s 42ms/step
    
<br>

## 7. 워드클라우드

```python
from wordcloud import WordCloud
import matplotlib.pyplot as plt

%matplotlib inline
def displayWordCloud(data = None, backgroundcolor = 'white', width=None, height=None):
    wordcloud = WordCloud(font_path = '/content/drive/MyDrive/sentence/MALGUN.TTF', 
                          background_color = backgroundcolor, 
                          width = width, 
                          height = height).generate(data)
    plt.figure(figsize = (15 , 10))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.show()

```

아래처럼 잘 만들어진다!



```python
displayWordCloud(data = ' '.join(words_list), width=600, height=400)
```


    
<img src="https://user-images.githubusercontent.com/115082062/213747331-24b21b29-5cb7-4581-9763-bcf6ef4d4d00.png">
    


원하는 모양에 맞추어 워드클라우드를 생성할 수도 있다. 간단하게 이미지를 지정해주고 넘파이 array로 바꾸어서 mask 파리미터에 넣어주면 된다.

나는 아령 사진을 넣었는데 아령처럼 생기진 않은 것 같다.


```python
from PIL import *
img = Image.open('/content/drive/MyDrive/sentence/아령.jpg')
imgArray = np.array(img)

%matplotlib inline
def displayWordCloud(data = None, backgroundcolor = 'white', width=None, height=None):
    wordcloud = WordCloud(font_path = '/content/drive/MyDrive/sentence/MALGUN.TTF', 
                          background_color = backgroundcolor, 
                          width = width, 
                          height = height,
                          mask=imgArray).generate(data)
    plt.figure(figsize = (15 , 10))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.show()

```


```python
displayWordCloud(data = ' '.join(words_list), width=600, height=400)
```


    
<img src="https://user-images.githubusercontent.com/115082062/213747320-b9d51f4d-8434-4963-9e3d-5918a7f05fbe.png">

<br>
    

