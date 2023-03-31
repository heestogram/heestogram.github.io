---
title: "[DL] GRU로 네이버 쇼핑 리뷰 감성분류"
excerpt: "keras의 GRU를 이용하여 네이버 쇼핑 리뷰를 전처리하고 감성 분류를 해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, RNN, LSTM, GRU, 시계열, keras]

published: true

categories:
  - DL

date: 2023-02-15 21:00:00
last_modified_at: 2023-02-15 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

## 1. 라이브러리 설치


```python
import os
import numpy as np
import pandas as pd
import tensorflow as tf
import matplotlib.pyplot as plt
import sklearn.preprocessing
from sklearn.metrics import r2_score

from keras.layers import Dense,Dropout,SimpleRNN,LSTM
from keras.models import Sequential
```


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
# mecab 설치

!git clone https://github.com/SOMJANG/Mecab-ko-for-Google-Colab.git
%cd Mecab-ko-for-Google-Colab
!bash install_mecab-ko_on_colab190912.sh
```

    Cloning into 'Mecab-ko-for-Google-Colab'...
    remote: Enumerating objects: 115, done.[K
    remote: Counting objects: 100% (24/24), done.[K
    remote: Compressing objects: 100% (20/20), done.[K
    remote: Total 115 (delta 11), reused 10 (delta 3), pack-reused 91[K
    Receiving objects: 100% (115/115), 1.27 MiB | 10.24 MiB/s, done.
    Resolving deltas: 100% (50/50), done.
    /content/Mecab-ko-for-Google-Colab
    Installing konlpy.....
    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting konlpy
      Downloading konlpy-0.6.0-py2.py3-none-any.whl (19.4 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m19.4/19.4 MB[0m [31m70.1 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting JPype1>=0.7.0
      Downloading JPype1-1.4.1-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (465 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m465.6/465.6 KB[0m [31m46.4 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: lxml>=4.1.0 in /usr/local/lib/python3.8/dist-packages (from konlpy) (4.9.2)
    Requirement already satisfied: numpy>=1.6 in /usr/local/lib/python3.8/dist-packages (from konlpy) (1.21.6)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from JPype1>=0.7.0->konlpy) (23.0)
    Installing collected packages: JPype1, konlpy
    Successfully installed JPype1-1.4.1 konlpy-0.6.0
    Done
    Installing mecab-0.996-ko-0.9.2.tar.gz.....
    Downloading mecab-0.996-ko-0.9.2.tar.gz.......
    from https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
    --2023-02-06 16:56:58--  https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
    Resolving bitbucket.org (bitbucket.org)... 104.192.141.1, 2406:da00:ff00::6b17:d1f5, 2406:da00:ff00::22c0:3470, ...
    Connecting to bitbucket.org (bitbucket.org)|104.192.141.1|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://bbuseruploads.s3.amazonaws.com/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-0.996-ko-0.9.2.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNEF262DGD&Signature=gmZiYsFNXB7X259Mn0bsqnPpY7c%3D&x-amz-security-token=FwoGZXIvYXdzENL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDLcg540wLoqDmxvwByK%2BAXPdA%2FF79RPhnAkQCvilIYMQ1n3ow07VFUUBS9lf%2BsNttFMpOoKiig8tiLfaL9vMQ8UgpLkLAFLVtKP43wO2o5RIcGnKo4zaqOOpQ39IKjmbTrAQ%2BuutOHzRRnwYvdlL%2Fz4VQw8jyiY8BMyXMjtT08CdUcq9O3TZ8zhFAfEd1FGs%2F9ZGR2tZdFHEan%2FnwGJHXPf5l%2FrbOwtTfXGC2o9cnheyF5DCCaNd%2Fj4l1vN96NiXCfjgaXSIyN7rdoXaoTYot%2BCEnwYyLevcB5Y8MCzIcFowuBTcOYzH4ho3MK9UBsLCzldh5ETyiHwQ3KGvXZeDLwfSgw%3D%3D&Expires=1675704128 [following]
    --2023-02-06 16:56:58--  https://bbuseruploads.s3.amazonaws.com/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-0.996-ko-0.9.2.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNEF262DGD&Signature=gmZiYsFNXB7X259Mn0bsqnPpY7c%3D&x-amz-security-token=FwoGZXIvYXdzENL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDLcg540wLoqDmxvwByK%2BAXPdA%2FF79RPhnAkQCvilIYMQ1n3ow07VFUUBS9lf%2BsNttFMpOoKiig8tiLfaL9vMQ8UgpLkLAFLVtKP43wO2o5RIcGnKo4zaqOOpQ39IKjmbTrAQ%2BuutOHzRRnwYvdlL%2Fz4VQw8jyiY8BMyXMjtT08CdUcq9O3TZ8zhFAfEd1FGs%2F9ZGR2tZdFHEan%2FnwGJHXPf5l%2FrbOwtTfXGC2o9cnheyF5DCCaNd%2Fj4l1vN96NiXCfjgaXSIyN7rdoXaoTYot%2BCEnwYyLevcB5Y8MCzIcFowuBTcOYzH4ho3MK9UBsLCzldh5ETyiHwQ3KGvXZeDLwfSgw%3D%3D&Expires=1675704128
    Resolving bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)... 54.231.235.17, 52.217.168.97, 52.217.102.140, ...
    Connecting to bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)|54.231.235.17|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 1414979 (1.3M) [application/x-tar]
    Saving to: ‘mecab-0.996-ko-0.9.2.tar.gz’
    
    mecab-0.996-ko-0.9. 100%[===================>]   1.35M  7.74MB/s    in 0.2s    
    
    2023-02-06 16:56:58 (7.74 MB/s) - ‘mecab-0.996-ko-0.9.2.tar.gz’ saved [1414979/1414979]
    
    Done
    Unpacking mecab-0.996-ko-0.9.2.tar.gz.......
    Done
    Change Directory to mecab-0.996-ko-0.9.2.......
    installing mecab-0.996-ko-0.9.2.tar.gz........
    configure
    make
    make check
    make install
    ldconfig
    Done
    Change Directory to /content
    Downloading mecab-ko-dic-2.1.1-20180720.tar.gz.......
    from https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.1.1-20180720.tar.gz
    --2023-02-06 16:58:39--  https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.1.1-20180720.tar.gz
    Resolving bitbucket.org (bitbucket.org)... 104.192.141.1, 2406:da00:ff00::22c0:3470, 2406:da00:ff00::6b17:d1f5, ...
    Connecting to bitbucket.org (bitbucket.org)|104.192.141.1|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://bbuseruploads.s3.amazonaws.com/a4fcd83e-34f1-454e-a6ac-c242c7d434d3/downloads/b5a0c703-7b64-45ed-a2d7-180e962710b6/mecab-ko-dic-2.1.1-20180720.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-ko-dic-2.1.1-20180720.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNB3YVVMGH&Signature=pCkkO8Ksf%2Fpgh519O%2Brn%2B6ejA1Y%3D&x-amz-security-token=FwoGZXIvYXdzENL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDHfFWQvswfni%2FckBmSK%2BAclElMhF5bHngDO2omgbiI7jhIxlsGYRHkczwupGkGjAuj0ir0WacqFA%2FCpzUyrTlSS1Lgbsd1r%2Bzxi%2FnELbylThFVboOeixxrF931Un7aaB3fTOuhmbVIdvTSxJ2dBFME8UqLxsY95wkNTXBvoET2M%2BykGm4kN1JGYTYuiGX2zvMujYpt%2BJqVmNWVPk7Glt1OVUDeGygzn2n7iQyu2rhJWadIGaRQVzESPMTibIzmtiICRZgc4KpvnMAHOaoD0ogN2EnwYyLWyvRMkCS%2BTyN0yqkk%2FmWP2yK1LrbT8RINzvoiIqmDpZS%2FsUGjBLEhSl3ueplg%3D%3D&Expires=1675703689 [following]
    --2023-02-06 16:58:39--  https://bbuseruploads.s3.amazonaws.com/a4fcd83e-34f1-454e-a6ac-c242c7d434d3/downloads/b5a0c703-7b64-45ed-a2d7-180e962710b6/mecab-ko-dic-2.1.1-20180720.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-ko-dic-2.1.1-20180720.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNB3YVVMGH&Signature=pCkkO8Ksf%2Fpgh519O%2Brn%2B6ejA1Y%3D&x-amz-security-token=FwoGZXIvYXdzENL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDHfFWQvswfni%2FckBmSK%2BAclElMhF5bHngDO2omgbiI7jhIxlsGYRHkczwupGkGjAuj0ir0WacqFA%2FCpzUyrTlSS1Lgbsd1r%2Bzxi%2FnELbylThFVboOeixxrF931Un7aaB3fTOuhmbVIdvTSxJ2dBFME8UqLxsY95wkNTXBvoET2M%2BykGm4kN1JGYTYuiGX2zvMujYpt%2BJqVmNWVPk7Glt1OVUDeGygzn2n7iQyu2rhJWadIGaRQVzESPMTibIzmtiICRZgc4KpvnMAHOaoD0ogN2EnwYyLWyvRMkCS%2BTyN0yqkk%2FmWP2yK1LrbT8RINzvoiIqmDpZS%2FsUGjBLEhSl3ueplg%3D%3D&Expires=1675703689
    Resolving bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)... 52.217.200.81, 52.216.112.243, 3.5.29.123, ...
    Connecting to bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)|52.217.200.81|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 49775061 (47M) [application/x-tar]
    Saving to: ‘mecab-ko-dic-2.1.1-20180720.tar.gz’
    
    mecab-ko-dic-2.1.1- 100%[===================>]  47.47M  69.5MB/s    in 0.7s    
    
    2023-02-06 16:58:40 (69.5 MB/s) - ‘mecab-ko-dic-2.1.1-20180720.tar.gz’ saved [49775061/49775061]
    
    Done
    Unpacking  mecab-ko-dic-2.1.1-20180720.tar.gz.......
    Done
    Change Directory to mecab-ko-dic-2.1.1-20180720
    Done
    installing........
    configure
    make
    make install
    apt-get update
    apt-get upgrade
    apt install curl
    apt install git
    bash <(curl -s https://raw.githubusercontent.com/konlpy/konlpy/master/scripts/mecab.sh)
    Done
    Successfully Installed
    Now you can use Mecab
    from konlpy.tag import Mecab
    mecab = Mecab()
    사용자 사전 추가 방법 : https://bit.ly/3k0ZH53
    NameError: name 'Tagger' is not defined 오류 발생 시 런타임을 재실행 해주세요
    블로그에 해결 방법을 남겨주신 tana님 감사합니다.
    


```python
import urllib.request
from collections import Counter
from konlpy.tag import Mecab
from sklearn.model_selection import train_test_split
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
```

<br>

## 2. 데이터 로드, train, test split


```python
urllib.request.urlretrieve("https://raw.githubusercontent.com/bab2min/corpus/master/sentiment/naver_shopping.txt", filename="ratings_total.txt")
```




    ('ratings_total.txt', <http.client.HTTPMessage at 0x7f94b1227be0>)




```python
total_data = pd.read_table('ratings_total.txt', names=['ratings','reviews'])    #column name 추가
print('전체 리뷰 개수: ', len(total_data))
```

    전체 리뷰 개수:  200000
    


```python
total_data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 200000 entries, 0 to 199999
    Data columns (total 2 columns):
     #   Column   Non-Null Count   Dtype 
    ---  ------   --------------   ----- 
     0   ratings  200000 non-null  int64 
     1   reviews  200000 non-null  object
    dtypes: int64(1), object(1)
    memory usage: 3.1+ MB
    


```python
total_data.groupby('ratings').count()
```





  <div id="df-baf06779-fbcd-4d3a-8894-4b52e7545110">
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
      <th>reviews</th>
    </tr>
    <tr>
      <th>ratings</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>36048</td>
    </tr>
    <tr>
      <th>2</th>
      <td>63989</td>
    </tr>
    <tr>
      <th>4</th>
      <td>18786</td>
    </tr>
    <tr>
      <th>5</th>
      <td>81177</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-baf06779-fbcd-4d3a-8894-4b52e7545110')"
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
          document.querySelector('#df-baf06779-fbcd-4d3a-8894-4b52e7545110 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-baf06779-fbcd-4d3a-8894-4b52e7545110');
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
# 평점 4,5인 리뷰에는 레이블 1 / 평점 1,2인 리뷰에는 레이블 0
total_data['label'] = np.select([total_data.ratings > 3], [1], default=0)
total_data[:5]
```





  <div id="df-b314d212-81bb-4039-838f-f62dfa758de6">
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
      <th>ratings</th>
      <th>reviews</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5</td>
      <td>배공빠르고 굿</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>택배가 엉망이네용 저희집 밑에층에 말도없이 놔두고가고</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>아주좋아요 바지 정말 좋아서2개 더 구매했어요 이가격에 대박입니다. 바느질이 조금 ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>선물용으로 빨리 받아서 전달했어야 하는 상품이었는데 머그컵만 와서 당황했습니다. 전...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>민트색상 예뻐요. 옆 손잡이는 거는 용도로도 사용되네요 ㅎㅎ</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-b314d212-81bb-4039-838f-f62dfa758de6')"
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
          document.querySelector('#df-b314d212-81bb-4039-838f-f62dfa758de6 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-b314d212-81bb-4039-838f-f62dfa758de6');
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
# 중복 샘플 제거
total_data.drop_duplicates(subset=['reviews'], inplace=True)
print('총 샘플의 수', len(total_data))
```

    총 샘플의 수 199908
    


```python
# null값 유무 확인

print(total_data.isnull().values.any())
```

    False
    


```python
# 3:1 비율로 train, test 나누기

train_data, test_data = train_test_split(total_data, test_size=.25, random_state=42)
print('훈련 리뷰 수 :', len(train_data))
print('테스트 리뷰 수 :', len(test_data))
```

    훈련 리뷰 수 : 149931
    테스트 리뷰 수 : 49977
    


```python
# 레이블 분포 확인
total_data.groupby('label').count()
```





  <div id="df-acd16a0d-f5f2-4911-9ad2-70bfc74bd567">
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
      <th>ratings</th>
      <th>reviews</th>
    </tr>
    <tr>
      <th>label</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>99955</td>
      <td>99955</td>
    </tr>
    <tr>
      <th>1</th>
      <td>99953</td>
      <td>99953</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-acd16a0d-f5f2-4911-9ad2-70bfc74bd567')"
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
          document.querySelector('#df-acd16a0d-f5f2-4911-9ad2-70bfc74bd567 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-acd16a0d-f5f2-4911-9ad2-70bfc74bd567');
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

## 3. 데이터 전처리, 형태소 분석


```python
train_data['reviews'] = train_data['reviews'].str.replace("[^ㄱ-ㅎ ㅏ-ㅣ 가-힣]","") #한글과 공백만 남기기기
train_data['reviews'].replace('', np.nan, inplace=True)
print(train_data.isnull().sum())
```

    <ipython-input-15-7eeb1b2c945b>:1: FutureWarning: The default value of regex will change from True to False in a future version.
      train_data['reviews'] = train_data['reviews'].str.replace("[^ㄱ-ㅎ ㅏ-ㅣ 가-힣]","") #한글과 공백만 남기기기
    

    ratings    0
    reviews    0
    label      0
    dtype: int64
    


```python
test_data.drop_duplicates(subset=['reviews'], inplace=True) 
test_data['reviews'] = test_data['reviews'].str.replace("[^ㄱ-ㅎㅏ-ㅣ가-힣 ]","")
test_data['reviews'].replace("",np.nan,inplace=True)
test_data = test_data.dropna(how='any')
print('전처리 후 테스트용 샘플 수: ', len(test_data))
```

    전처리 후 테스트용 샘플 수:  49977
    

    <ipython-input-16-c6b4042129ef>:2: FutureWarning: The default value of regex will change from True to False in a future version.
      test_data['reviews'] = test_data['reviews'].str.replace("[^ㄱ-ㅎㅏ-ㅣ가-힣 ]","")
    


```python
mecab = Mecab()
stopwords = ['도', '는', '다', '의', '가', '이', '은', '한', '에', '하', '고', '을', '를', '인', '듯', '과', '와', '네', '들', '듯', '지', '임', '게']
```


```python
train_data['tokenized'] = train_data['reviews'].apply(mecab.morphs)
train_data['tokenized'] = train_data['tokenized'].apply(lambda x: [s for s in x if s not in stopwords])

test_data['tokenized'] = test_data['reviews'].apply(mecab.morphs)
test_data['tokenized'] = test_data['reviews'].apply(lambda x: [s for s in x if s not in stopwords])
```


```python
train_data['tokenized']
```




    59666     [사이즈, 센치, 씩, 늘린, 건데, 작, 아요, 그리고, 색상, 완전, 달라요, ...
    12433         [ㅂ, 불, 만족, 빗이, 아픔, 멍, 피부, 빗, 질, 못해, 주, 겟, 네요]
    146516    [제품, 쓰, 삼, 일, 만, 변기, 물, 잘, 안, 내려갔, 어요, 혹시나, 해서...
    158109                                        [적당, 만족, 합니다]
    70219      [편하, 자고, 이용, 밀키, 튼, 데, 손, 은근, 많이, 서, 저, 패, 쓰, 요]
                                    ...                        
    119904    [그냥, 그래요, ㄷ, ㄷ, ㄷ, ㄷ, ㅂ, ㅂ, ㅂ, ㅂ, 그냥, 그래요, ㄷ, ...
    103714    [비싸, 요, 진짜, 별거, 아니, 허접, 생겼, 는데, 이게, 만, 원, 라니, ...
    131960                           [장, 주문, 안, 됩니다, 장, 가능, 해요]
    146908    [하림, 치킨, 여기, 서, 구입, 니, 엄청, 저렴, 네요, 배송, 쾅, 꽝, 얼...
    121984                       [조금, 약해, 보이, 는데, 저렴, 잘, 삿, 어요]
    Name: tokenized, Length: 149931, dtype: object


<br>

## 4. 단어, 길이의 분포 확인


```python
negative_words = np.hstack(train_data[train_data.label==0]['tokenized'].values)
positive_words = np.hstack(train_data[train_data.label==1]['tokenized'].values)
```


```python
negative_word_count = Counter(negative_words)
print(negative_word_count.most_common(20))
```

    [('네요', 31799), ('는데', 20295), ('안', 19718), ('어요', 14849), ('있', 13200), ('너무', 13058), ('했', 11783), ('좋', 9812), ('배송', 9677), ('같', 8997), ('구매', 8876), ('어', 8869), ('거', 8854), ('없', 8670), ('아요', 8642), ('습니다', 8436), ('그냥', 8355), ('되', 8345), ('잘', 8029), ('않', 7984)]
    


```python
positive_word_count = Counter(positive_words)
print(positive_word_count.most_common(20))
```

    [('좋', 39488), ('아요', 21184), ('네요', 19895), ('어요', 18686), ('잘', 18602), ('구매', 16171), ('습니다', 13320), ('있', 12391), ('배송', 12275), ('는데', 11670), ('했', 9818), ('합니다', 9801), ('먹', 9635), ('재', 9273), ('너무', 8397), ('같', 7868), ('만족', 7261), ('거', 6482), ('어', 6294), ('쓰', 6292)]
    


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


```python
displayWordCloud(data = ' '.join(negative_words), width=600, height=400)
```


    
<img src="https://user-images.githubusercontent.com/115082062/229086634-a13ed41d-9a46-45e0-b6cd-8af3b861a6c0.png">

    

<br>

```python
displayWordCloud(data = ' '.join(positive_words), width=600, height=400)
```


    
<img src="https://user-images.githubusercontent.com/115082062/229086804-7f4756c1-955a-4694-a4ac-0911b25a9e6d.png">

    



```python
fig, (ax1,ax2) = plt.subplots(1,2,figsize=(10,5))
text_len = train_data[train_data['label']==1]['tokenized'].map(lambda x:len(x)) # 각 row의 길이를 반환
ax1.hist(text_len, color='red')
ax1.set_title('Positive Reviews')
ax1.set_xlabel('length of samples')
ax1.set_ylabel('number of samples')
print('긍정 리뷰 평균 길이: ', np.mean(text_len))

text_len = train_data[train_data['label']==0]['tokenized'].map(lambda x:len(x))
ax2.hist(text_len, color='blue')
ax2.set_title('Negative Reviews')
fig.suptitle('Words in texts')
ax2.set_xlabel('length of samples')
ax2.set_ylabel('number of samples')
print('부정 리뷰의 평균 길이 :', np.mean(text_len))

plt.show()
```

    긍정 리뷰 평균 길이:  13.587751456414221
    부정 리뷰의 평균 길이 : 17.029525614672043
    


    
<img src="https://user-images.githubusercontent.com/115082062/229086887-1b2e0e70-657c-4eb1-bb30-8de9a775f52b.png">

<br>
    



```python
X_train = train_data['tokenized'].values
y_train = train_data['label'].values
X_test= test_data['tokenized'].values
y_test = test_data['label'].values
```

<br>

## 5. 토크나이징


```python
tokenizer = Tokenizer()
tokenizer.fit_on_texts(X_train)
```


```python
threshold=2
total_cnt = len(tokenizer.word_index) #vocab수
rare_cnt=0  #등장빈도 threshold 미만인 단어 수 카운트
total_freq =0   #훈련 데이터의 전체 단어 빈도수 총합
rare_freq = 0   #등장빈도 threshold 미만인 단어의 등장 빈도수 총합

for key, value in tokenizer.word_counts.items(): # items에 각 단어가 ('단어', 출현 빈도) 형식으로 저장되어있다.
    total_freq += value # sum(모든 단어의 출현빈도)

    if (value < threshold): # 특정 단어의 출현 빈도가 기준값(2)보다 작으면
        rare_cnt += 1 # 희귀 단어 count를 하나 올리고
        rare_freq += value # 희귀 단어 출현 빈도에 그 단어의 출현 빈도만큼 더한다.

print('단어 집합(vocabulary)의 크기 :',total_cnt)
print('등장빈도가 %s번 이하인 희귀 단어의 수: %s' %(threshold-1, rare_cnt))
print('단어 집합에서 희귀 단어의 비율:', (rare_cnt/total_cnt)*100)
print('전체 등장 빈도에서 희귀 단어 등장 빈도 비율:', (rare_freq/total_freq)*100)
```

    단어 집합(vocabulary)의 크기 : 39997
    등장빈도가 1번 이하인 희귀 단어의 수: 18212
    단어 집합에서 희귀 단어의 비율: 45.53341500612546
    전체 등장 빈도에서 희귀 단어 등장 빈도 비율: 0.79352492030765
    


```python
# 전체 단어 중 등장 빈도 2 미만인 단어 제거할 것
vocab_size = total_cnt - rare_cnt + 2 #0번 패딩 토큰, OOV토큰 고려해서 +2
print("단어 집합 크기:", vocab_size)
```

    단어 집합 크기: 21787
    


```python
'''토크나이저로 텍스트 시퀀스를 숫자 시퀀스로 변환
   정수 인코딩에서 이보다 큰 숫자 부여된 단어는 OOV로 변환'''

tokenizer = Tokenizer(vocab_size, oov_token='OOV')
tokenizer.fit_on_texts(X_train)
X_train = tokenizer.texts_to_sequences(X_train)
X_test = tokenizer.texts_to_sequences(X_test)
```


```python
# 정수 인코딩된 단어들이 시퀀스로 변환되었음음
print(X_train[:3])
print(X_test[:3])
```

    [[67, 2060, 299, 14260, 263, 73, 6, 236, 168, 137, 805, 2951, 625, 2, 77, 62, 207, 40, 1343, 155, 3, 6], [482, 409, 52, 8530, 2561, 2517, 339, 2918, 250, 2357, 38, 473, 2], [46, 24, 825, 105, 35, 2372, 160, 7, 10, 8061, 4, 1319, 29, 140, 322, 41, 59, 160, 140, 7, 1916, 2, 113, 162, 1379, 323, 119, 136]]
    [[721, 581, 1, 704, 1, 767, 1, 116, 1, 434, 1832, 522, 3516, 1935, 59], [956, 991, 1, 1, 1302, 1, 113, 1, 640, 56, 22], [410, 3327, 1, 1274, 2079, 22, 1, 5418, 290, 134, 1, 3, 27, 1, 15, 28, 22, 1, 513, 1, 449, 17, 33, 1, 618, 1342, 1, 33, 59, 1, 7, 1, 22]]
    

<br>

## 6. 패딩


```python
print("리뷰 최대 길이: ", max(len(l) for l in X_train))
print("리뷰 평균 길이: ", sum(map(len, X_train))/len(X_train))
```

    리뷰 최대 길이:  85
    리뷰 평균 길이:  15.30754813881052
    


```python
plt.hist([len(s) for s in X_train], bins=50)
plt.xlabel('length of samples')
plt.ylabel('number of samles')
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229087006-68311587-6007-4aca-b9d1-eb63dcdfb76d.png">

    



```python
def below_threshold_len(max_len, nested_list):
    cnt=0
    for s in nested_list:
        if(len(s) <= max_len):
            cnt += 1
    print("전체 샘플 중 길이가 %s 이하인 샘플의 비율: %s" %(max_len, (cnt/len(nested_list))*100))
```


```python
max_len = 55
below_threshold_len(max_len, X_train)
```

    전체 샘플 중 길이가 55 이하인 샘플의 비율: 99.8459291273986
    


```python
# 길이 55로 패딩
X_train = pad_sequences(X_train, maxlen = max_len)
X_test = pad_sequences(X_test, maxlen = max_len)
```

<br>

## 7. GRU로 감성분류

코랩 런타임 유형을 GPU로 바꾸고 실행했다.


```python
from keras.layers import Embedding, Dense, GRU, Dropout, Activation
from keras.models import Sequential, load_model
from keras.callbacks import EarlyStopping, ModelCheckpoint
```


```python
gru_model = Sequential()
gru_model.add(Embedding(input_dim= vocab_size,output_dim= 100))
gru_model.add(GRU(128))
gru_model.add(Dense(1, activation='sigmoid'))

es = EarlyStopping(monitor='val_loss', mode='min', verbose=1, patience=4)
mc = ModelCheckpoint('best_model.h5', monitor='val_acc', mode='max', verbose=1, save_best_only=True)

gru_model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])

history = gru_model.fit(X_train, y_train, epochs=15, callbacks=[es,mc], batch_size=60, validation_split=.2)
```

    Epoch 1/15
    1994/2000 [============================>.] - ETA: 0s - loss: 0.2725 - acc: 0.8971
    Epoch 1: val_acc improved from -inf to 0.91833, saving model to best_model.h5
    2000/2000 [==============================] - 24s 9ms/step - loss: 0.2725 - acc: 0.8971 - val_loss: 0.2285 - val_acc: 0.9183
    Epoch 2/15
    1995/2000 [============================>.] - ETA: 0s - loss: 0.2148 - acc: 0.9230
    Epoch 2: val_acc did not improve from 0.91833
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.2147 - acc: 0.9230 - val_loss: 0.2326 - val_acc: 0.9133
    Epoch 3/15
    2000/2000 [==============================] - ETA: 0s - loss: 0.1983 - acc: 0.9293
    Epoch 3: val_acc improved from 0.91833 to 0.92767, saving model to best_model.h5
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.1983 - acc: 0.9293 - val_loss: 0.2028 - val_acc: 0.9277
    Epoch 4/15
    1997/2000 [============================>.] - ETA: 0s - loss: 0.1872 - acc: 0.9336
    Epoch 4: val_acc did not improve from 0.92767
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.1873 - acc: 0.9335 - val_loss: 0.2142 - val_acc: 0.9208
    Epoch 5/15
    1996/2000 [============================>.] - ETA: 0s - loss: 0.1782 - acc: 0.9372
    Epoch 5: val_acc did not improve from 0.92767
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.1781 - acc: 0.9372 - val_loss: 0.2068 - val_acc: 0.9251
    Epoch 6/15
    2000/2000 [==============================] - ETA: 0s - loss: 0.1689 - acc: 0.9408
    Epoch 6: val_acc did not improve from 0.92767
    2000/2000 [==============================] - 19s 9ms/step - loss: 0.1689 - acc: 0.9408 - val_loss: 0.2191 - val_acc: 0.9218
    Epoch 7/15
    1998/2000 [============================>.] - ETA: 0s - loss: 0.1597 - acc: 0.9442
    Epoch 7: val_acc did not improve from 0.92767
    2000/2000 [==============================] - 19s 9ms/step - loss: 0.1597 - acc: 0.9442 - val_loss: 0.2137 - val_acc: 0.9232
    Epoch 7: early stopping
    


```python
gru_model = Sequential()
gru_model.add(Embedding(input_dim= vocab_size,output_dim= 100))
gru_model.add(GRU(128))
gru_model.add(Activation('tanh'))
gru_model.add(Dropout(0.2))
gru_model.add(Dense(1, activation='relu'))

es = EarlyStopping(monitor='val_loss', mode='min', verbose=1, patience=4)
mc = ModelCheckpoint('best_model.h5', monitor='val_acc', mode='max', verbose=1, save_best_only=True)

gru_model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])

history = gru_model.fit(X_train, y_train, epochs=15, callbacks=[es,mc], batch_size=60, validation_split=.2)
```

    Epoch 1/15
    1998/2000 [============================>.] - ETA: 0s - loss: 0.5146 - acc: 0.8783
    Epoch 1: val_acc improved from -inf to 0.90232, saving model to best_model.h5
    2000/2000 [==============================] - 20s 9ms/step - loss: 0.5147 - acc: 0.8783 - val_loss: 0.7252 - val_acc: 0.9023
    Epoch 2/15
    1997/2000 [============================>.] - ETA: 0s - loss: 0.4555 - acc: 0.9074
    Epoch 2: val_acc improved from 0.90232 to 0.91166, saving model to best_model.h5
    2000/2000 [==============================] - 19s 10ms/step - loss: 0.4553 - acc: 0.9074 - val_loss: 0.5507 - val_acc: 0.9117
    Epoch 3/15
    1999/2000 [============================>.] - ETA: 0s - loss: 0.4140 - acc: 0.9159
    Epoch 3: val_acc improved from 0.91166 to 0.92063, saving model to best_model.h5
    2000/2000 [==============================] - 17s 9ms/step - loss: 0.4140 - acc: 0.9159 - val_loss: 0.4179 - val_acc: 0.9206
    Epoch 4/15
    1994/2000 [============================>.] - ETA: 0s - loss: 0.4019 - acc: 0.9226
    Epoch 4: val_acc improved from 0.92063 to 0.92293, saving model to best_model.h5
    2000/2000 [==============================] - 17s 9ms/step - loss: 0.4015 - acc: 0.9226 - val_loss: 0.3970 - val_acc: 0.9229
    Epoch 5/15
    1998/2000 [============================>.] - ETA: 0s - loss: 0.3799 - acc: 0.9271
    Epoch 5: val_acc did not improve from 0.92293
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.3801 - acc: 0.9270 - val_loss: 0.3479 - val_acc: 0.9183
    Epoch 6/15
    1995/2000 [============================>.] - ETA: 0s - loss: 0.3788 - acc: 0.9310
    Epoch 6: val_acc did not improve from 0.92293
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.3787 - acc: 0.9311 - val_loss: 0.4709 - val_acc: 0.9204
    Epoch 7/15
    1998/2000 [============================>.] - ETA: 0s - loss: 0.3747 - acc: 0.9337
    Epoch 7: val_acc did not improve from 0.92293
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.3748 - acc: 0.9337 - val_loss: 0.3532 - val_acc: 0.9122
    Epoch 8/15
    1995/2000 [============================>.] - ETA: 0s - loss: 0.3463 - acc: 0.9373
    Epoch 8: val_acc did not improve from 0.92293
    2000/2000 [==============================] - 17s 9ms/step - loss: 0.3463 - acc: 0.9373 - val_loss: 0.3703 - val_acc: 0.9194
    Epoch 9/15
    1999/2000 [============================>.] - ETA: 0s - loss: 0.3580 - acc: 0.9399
    Epoch 9: val_acc did not improve from 0.92293
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.3580 - acc: 0.9399 - val_loss: 0.4435 - val_acc: 0.9219
    Epoch 9: early stopping
    


```python
gru_model = Sequential()
gru_model.add(Embedding(input_dim= vocab_size,output_dim= 256))
gru_model.add(GRU(128))
gru_model.add(Dropout(0.3))
gru_model.add(Dense(1, activation='sigmoid'))

es = EarlyStopping(monitor='val_loss', mode='min', verbose=1, patience=4)
mc = ModelCheckpoint('best_model.h5', monitor='val_acc', mode='max', verbose=1, save_best_only=True)

gru_model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])

history = gru_model.fit(X_train, y_train, epochs=15, callbacks=[es,mc], batch_size=60, validation_split=.2)
```

    Epoch 1/15
    1996/2000 [============================>.] - ETA: 0s - loss: 0.2660 - acc: 0.9005
    Epoch 1: val_acc improved from -inf to 0.91356, saving model to best_model.h5
    2000/2000 [==============================] - 19s 9ms/step - loss: 0.2660 - acc: 0.9005 - val_loss: 0.2375 - val_acc: 0.9136
    Epoch 2/15
    1995/2000 [============================>.] - ETA: 0s - loss: 0.2119 - acc: 0.9236
    Epoch 2: val_acc improved from 0.91356 to 0.92437, saving model to best_model.h5
    2000/2000 [==============================] - 17s 8ms/step - loss: 0.2118 - acc: 0.9236 - val_loss: 0.2077 - val_acc: 0.9244
    Epoch 3/15
    2000/2000 [==============================] - ETA: 0s - loss: 0.1949 - acc: 0.9311
    Epoch 3: val_acc improved from 0.92437 to 0.92747, saving model to best_model.h5
    2000/2000 [==============================] - 22s 11ms/step - loss: 0.1949 - acc: 0.9311 - val_loss: 0.2061 - val_acc: 0.9275
    Epoch 4/15
    1999/2000 [============================>.] - ETA: 0s - loss: 0.1828 - acc: 0.9361
    Epoch 4: val_acc improved from 0.92747 to 0.92787, saving model to best_model.h5
    2000/2000 [==============================] - 17s 9ms/step - loss: 0.1828 - acc: 0.9361 - val_loss: 0.2009 - val_acc: 0.9279
    Epoch 5/15
    1993/2000 [============================>.] - ETA: 0s - loss: 0.1716 - acc: 0.9406
    Epoch 5: val_acc did not improve from 0.92787
    2000/2000 [==============================] - 17s 8ms/step - loss: 0.1717 - acc: 0.9405 - val_loss: 0.2026 - val_acc: 0.9265
    Epoch 6/15
    1998/2000 [============================>.] - ETA: 0s - loss: 0.1619 - acc: 0.9444
    Epoch 6: val_acc did not improve from 0.92787
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.1619 - acc: 0.9444 - val_loss: 0.2105 - val_acc: 0.9251
    Epoch 7/15
    1997/2000 [============================>.] - ETA: 0s - loss: 0.1515 - acc: 0.9484
    Epoch 7: val_acc did not improve from 0.92787
    2000/2000 [==============================] - 18s 9ms/step - loss: 0.1515 - acc: 0.9485 - val_loss: 0.2094 - val_acc: 0.9233
    Epoch 8/15
    1997/2000 [============================>.] - ETA: 0s - loss: 0.1420 - acc: 0.9517
    Epoch 8: val_acc did not improve from 0.92787
    2000/2000 [==============================] - 17s 8ms/step - loss: 0.1420 - acc: 0.9517 - val_loss: 0.2217 - val_acc: 0.9212
    Epoch 8: early stopping
    


```python
# epoch별 정확도 그래프
plt.plot(history.history['acc'])
plt.plot(history.history['val_acc'])
plt.title('model accuracy')
plt.ylabel('accuracy')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()

#epoch별 loss 그래프
plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('model loss')
plt.ylabel('loss')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229087082-7edbd1f1-2c68-4836-a094-1ef2d70c5f44.png">

    



    
<img src="https://user-images.githubusercontent.com/115082062/229087227-dda7fab4-6e9a-4949-a733-16d72a2e85b3.png">

<br>
    

