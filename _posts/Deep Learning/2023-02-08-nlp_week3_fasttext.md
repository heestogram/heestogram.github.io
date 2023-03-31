---
title: "[NLP] FastText와 Word2Vec 비교"
excerpt: "Word2Vec와 FastText로 각각 토큰화를 하고 단어 유사도를 비교해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, Word2Vec, FastText]

published: true

categories:
  - DL

date: 2023-02-08 21:00:00
last_modified_at: 2023-02-08 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

단어를 벡터로 만드는 방법에는 Word2Vec를 제외하고도 페이스북에서 개발한 FastText가 있다. Word2Vec는 단어를 쪼개질 수 없는 최소 단위로 보는 반면, FastText는 하나의 단어 안에도 여러 단어(subword)가 존재하는 것으로 간주한다.

<br>

```python
# Colab에 Mecab 설치
!git clone https://github.com/SOMJANG/Mecab-ko-for-Google-Colab.git
%cd Mecab-ko-for-Google-Colab
!bash install_mecab-ko_on_colab190912.sh
```

    Cloning into 'Mecab-ko-for-Google-Colab'...
    remote: Enumerating objects: 115, done.[K
    remote: Counting objects: 100% (24/24), done.[K
    remote: Compressing objects: 100% (20/20), done.[K
    remote: Total 115 (delta 11), reused 10 (delta 3), pack-reused 91[K
    Receiving objects: 100% (115/115), 1.27 MiB | 18.07 MiB/s, done.
    Resolving deltas: 100% (50/50), done.
    /content/Mecab-ko-for-Google-Colab
    Installing konlpy.....
    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting konlpy
      Downloading konlpy-0.6.0-py2.py3-none-any.whl (19.4 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m19.4/19.4 MB[0m [31m31.7 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: lxml>=4.1.0 in /usr/local/lib/python3.8/dist-packages (from konlpy) (4.9.2)
    Collecting JPype1>=0.7.0
      Downloading JPype1-1.4.1-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (465 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m465.6/465.6 KB[0m [31m17.6 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: numpy>=1.6 in /usr/local/lib/python3.8/dist-packages (from konlpy) (1.21.6)
    Requirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from JPype1>=0.7.0->konlpy) (21.3)
    Requirement already satisfied: pyparsing!=3.0.5,>=2.0.2 in /usr/local/lib/python3.8/dist-packages (from packaging->JPype1>=0.7.0->konlpy) (3.0.9)
    Installing collected packages: JPype1, konlpy
    Successfully installed JPype1-1.4.1 konlpy-0.6.0
    Done
    Installing mecab-0.996-ko-0.9.2.tar.gz.....
    Downloading mecab-0.996-ko-0.9.2.tar.gz.......
    from https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
    --2023-01-27 10:15:34--  https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
    Resolving bitbucket.org (bitbucket.org)... 18.205.93.2, 18.205.93.1, 18.205.93.0, ...
    Connecting to bitbucket.org (bitbucket.org)|18.205.93.2|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://bbuseruploads.s3.amazonaws.com/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-0.996-ko-0.9.2.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNIAS3ZMGW&Signature=0WUMbbdT48DyfImAvrTDLIk0ZtA%3D&x-amz-security-token=FwoGZXIvYXdzENz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDD4hMmDklksPMTTyziK%2BAUUD3OmgCCAFAPOqX7zWbgf%2BFp%2FZS%2Bz1uP4%2B0p5pFUcOJkaSf2U8rkITGbVSxeAiIjsB3mJJzhQu7slIHU36c4NRmH2N3YrjkgVmzaUnNxGt7bW1lxe87Nusvovea6Qn8i3cAUN3gkO0fjhbKIUwDH%2B1%2FXe0E%2FvDYo7xHIE7MoP0ztzOGCj8n%2Ft0oJ7aKfbTrGsMl74bIfHKjHz5KTjpkpUdt77a5GQc4ZEED1lcn4G%2FHbsh06TCDbXUfmxmUqMotsjOngYyLZYNUgKCavxqol4Sy9%2FFytXxC8NGheFJFzRhwlM05Bdre3Knczcl%2BQkIt6zjkg%3D%3D&Expires=1674816318 [following]
    --2023-01-27 10:15:34--  https://bbuseruploads.s3.amazonaws.com/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-0.996-ko-0.9.2.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNIAS3ZMGW&Signature=0WUMbbdT48DyfImAvrTDLIk0ZtA%3D&x-amz-security-token=FwoGZXIvYXdzENz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDD4hMmDklksPMTTyziK%2BAUUD3OmgCCAFAPOqX7zWbgf%2BFp%2FZS%2Bz1uP4%2B0p5pFUcOJkaSf2U8rkITGbVSxeAiIjsB3mJJzhQu7slIHU36c4NRmH2N3YrjkgVmzaUnNxGt7bW1lxe87Nusvovea6Qn8i3cAUN3gkO0fjhbKIUwDH%2B1%2FXe0E%2FvDYo7xHIE7MoP0ztzOGCj8n%2Ft0oJ7aKfbTrGsMl74bIfHKjHz5KTjpkpUdt77a5GQc4ZEED1lcn4G%2FHbsh06TCDbXUfmxmUqMotsjOngYyLZYNUgKCavxqol4Sy9%2FFytXxC8NGheFJFzRhwlM05Bdre3Knczcl%2BQkIt6zjkg%3D%3D&Expires=1674816318
    Resolving bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)... 54.231.192.33, 52.216.52.209, 52.216.21.99, ...
    Connecting to bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)|54.231.192.33|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 1414979 (1.3M) [application/x-tar]
    Saving to: ‘mecab-0.996-ko-0.9.2.tar.gz’
    
    mecab-0.996-ko-0.9. 100%[===================>]   1.35M  --.-KB/s    in 0.1s    
    
    2023-01-27 10:15:34 (10.6 MB/s) - ‘mecab-0.996-ko-0.9.2.tar.gz’ saved [1414979/1414979]
    
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
    --2023-01-27 10:17:37--  https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.1.1-20180720.tar.gz
    Resolving bitbucket.org (bitbucket.org)... 18.205.93.0, 18.205.93.1, 18.205.93.2, ...
    Connecting to bitbucket.org (bitbucket.org)|18.205.93.0|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://bbuseruploads.s3.amazonaws.com/a4fcd83e-34f1-454e-a6ac-c242c7d434d3/downloads/b5a0c703-7b64-45ed-a2d7-180e962710b6/mecab-ko-dic-2.1.1-20180720.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-ko-dic-2.1.1-20180720.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNK4SIQU6I&Signature=1jG1BiE8GwyaC7KNybHLzRPX3dM%3D&x-amz-security-token=FwoGZXIvYXdzENz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDKnCJyrwF6mc29OJqSK%2BAbdOfQhkWa1uu3r2GI9cxBqLNKkrkc05K68f0MSxveMc5p01qX685HDcIl7%2FCty5a46840zh7oaND1UEo%2FjrT%2BRiQTyFbDwlQvfwaqyxvduhAIA1OhhtAQA%2F6nHxN0Ub7s0e%2B6pAOBrrx6vHWGpItmHTEEDm0lF9wtqZsd%2BPzbeseQeUwkwmZ4CE1ZIZNG5RiT2qieeKFsONUPysD%2F%2BGxCH3xxreGm7eqzLGG66%2FwefIMmKn%2BzXwVQoH0KujiSMorcnOngYyLQDyLEgg2uVbjyw0eb1lYQX%2Bo3JavpYn5ucsBil9FcoEim%2FMUx3QWKB9GEsPlg%3D%3D&Expires=1674816437 [following]
    --2023-01-27 10:17:37--  https://bbuseruploads.s3.amazonaws.com/a4fcd83e-34f1-454e-a6ac-c242c7d434d3/downloads/b5a0c703-7b64-45ed-a2d7-180e962710b6/mecab-ko-dic-2.1.1-20180720.tar.gz?response-content-disposition=attachment%3B%20filename%3D%22mecab-ko-dic-2.1.1-20180720.tar.gz%22&response-content-encoding=None&AWSAccessKeyId=ASIA6KOSE3BNK4SIQU6I&Signature=1jG1BiE8GwyaC7KNybHLzRPX3dM%3D&x-amz-security-token=FwoGZXIvYXdzENz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDKnCJyrwF6mc29OJqSK%2BAbdOfQhkWa1uu3r2GI9cxBqLNKkrkc05K68f0MSxveMc5p01qX685HDcIl7%2FCty5a46840zh7oaND1UEo%2FjrT%2BRiQTyFbDwlQvfwaqyxvduhAIA1OhhtAQA%2F6nHxN0Ub7s0e%2B6pAOBrrx6vHWGpItmHTEEDm0lF9wtqZsd%2BPzbeseQeUwkwmZ4CE1ZIZNG5RiT2qieeKFsONUPysD%2F%2BGxCH3xxreGm7eqzLGG66%2FwefIMmKn%2BzXwVQoH0KujiSMorcnOngYyLQDyLEgg2uVbjyw0eb1lYQX%2Bo3JavpYn5ucsBil9FcoEim%2FMUx3QWKB9GEsPlg%3D%3D&Expires=1674816437
    Resolving bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)... 3.5.29.124, 52.216.52.153, 52.217.72.68, ...
    Connecting to bbuseruploads.s3.amazonaws.com (bbuseruploads.s3.amazonaws.com)|3.5.29.124|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 49775061 (47M) [application/x-tar]
    Saving to: ‘mecab-ko-dic-2.1.1-20180720.tar.gz’
    
    mecab-ko-dic-2.1.1- 100%[===================>]  47.47M   103MB/s    in 0.5s    
    
    2023-01-27 10:17:38 (103 MB/s) - ‘mecab-ko-dic-2.1.1-20180720.tar.gz’ saved [49775061/49775061]
    
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
from konlpy.tag import Mecab
mecab = Mecab()
```


```python
# 한글 자모 단위 처리 패키지 설치
!pip install hgtk
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting hgtk
      Downloading hgtk-0.1.3.tar.gz (6.2 kB)
      Preparing metadata (setup.py) ... [?25l[?25hdone
    Building wheels for collected packages: hgtk
      Building wheel for hgtk (setup.py) ... [?25l[?25hdone
      Created wheel for hgtk: filename=hgtk-0.1.3-py2.py3-none-any.whl size=6688 sha256=ccb728f3e9ec15bf590a90a00872c21a0b6d03e6fd313964f1798f63864a7ce7
      Stored in directory: /root/.cache/pip/wheels/93/33/b8/bc2256172a415340e34f3c11ef2b0f3f391769000bb74de988
    Successfully built hgtk
    Installing collected packages: hgtk
    Successfully installed hgtk-0.1.3
    


```python
# fasttext 설치
!git clone https://github.com/facebookresearch/fastText.git
%cd fastText
!make
!pip install .
```

    Cloning into 'fastText'...
    remote: Enumerating objects: 3930, done.[K
    remote: Counting objects: 100% (944/944), done.[K
    remote: Compressing objects: 100% (140/140), done.[K
    remote: Total 3930 (delta 854), reused 804 (delta 804), pack-reused 2986[K
    Receiving objects: 100% (3930/3930), 8.24 MiB | 15.81 MiB/s, done.
    Resolving deltas: 100% (2505/2505), done.
    /content/Mecab-ko-for-Google-Colab/fastText
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/args.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/autotune.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/matrix.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/dictionary.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/loss.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/productquantizer.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/densematrix.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/quantmatrix.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/vector.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/model.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/utils.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/meter.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG -c src/fasttext.cc
    c++ -pthread -std=c++11 -march=native -O3 -funroll-loops -DNDEBUG args.o autotune.o matrix.o dictionary.o loss.o productquantizer.o densematrix.o quantmatrix.o vector.o model.o utils.o meter.o fasttext.o src/main.cc -o fasttext
    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Processing /content/Mecab-ko-for-Google-Colab/fastText
      Preparing metadata (setup.py) ... [?25l[?25hdone
    Collecting pybind11>=2.2
      Using cached pybind11-2.10.3-py3-none-any.whl (222 kB)
    Requirement already satisfied: setuptools>=0.7.0 in /usr/local/lib/python3.8/dist-packages (from fasttext==0.9.2) (57.4.0)
    Requirement already satisfied: numpy in /usr/local/lib/python3.8/dist-packages (from fasttext==0.9.2) (1.21.6)
    Building wheels for collected packages: fasttext
      Building wheel for fasttext (setup.py) ... [?25l[?25hdone
      Created wheel for fasttext: filename=fasttext-0.9.2-cp38-cp38-linux_x86_64.whl size=4393595 sha256=56a469331166df0f3d88aab254d9b878cc8a896582e6b59875992128e70d2470
      Stored in directory: /tmp/pip-ephem-wheel-cache-o2baqkp6/wheels/3b/a9/65/8f58628dd4d195a5073adc57ce4877b6a22bd3fab7047c6984
    Successfully built fasttext
    Installing collected packages: pybind11, fasttext
    Successfully installed fasttext-0.9.2 pybind11-2.10.3
    

<br>

## 1. 데이터 로드


```python
import re
import pandas as pd
import urllib.request
from tqdm import tqdm
import hgtk
```


```python
# 네이버 쇼핑 리뷰
urllib.request.urlretrieve("https://raw.githubusercontent.com/bab2min/corpus/master/sentiment/naver_shopping.txt", filename="ratings_total.txt")
```




    ('ratings_total.txt', <http.client.HTTPMessage at 0x7f39131fd9a0>)




```python
total_data = pd.read_table('ratings_total.txt', names=['ratings', 'reviews'])
print('전체 리뷰 개수 :',len(total_data)) # 전체 리뷰 개수 출력
```

    전체 리뷰 개수 : 200000
    


```python
total_data.head()
```





  <div id="df-76a82664-bee2-4748-9a74-c100e7148340">
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
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5</td>
      <td>배공빠르고 굿</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>택배가 엉망이네용 저희집 밑에층에 말도없이 놔두고가고</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>아주좋아요 바지 정말 좋아서2개 더 구매했어요 이가격에 대박입니다. 바느질이 조금 ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>선물용으로 빨리 받아서 전달했어야 하는 상품이었는데 머그컵만 와서 당황했습니다. 전...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>민트색상 예뻐요. 옆 손잡이는 거는 용도로도 사용되네요 ㅎㅎ</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-76a82664-bee2-4748-9a74-c100e7148340')"
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
          document.querySelector('#df-76a82664-bee2-4748-9a74-c100e7148340 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-76a82664-bee2-4748-9a74-c100e7148340');
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


## 2. hgtk 튜토리얼

word embedding이 단어 단위의 임베딩이었다면, character embedding은 문자 단위의 임베딩이다. 한국어를 character embedding할 수 있는 것이 바로 자음 모음 분리기 hgtk이다.

 영어는 하나의 알파벳(52자)를 기준으로 character embedding을 하지만, 한국어에서 하나의 음절별로 character embedding을 하면 11172개의 음절이 있기 때문에 계산량이 너무 많다. 그래서 그보다 작은 단위인 자음 모음으로 분리하는 것이다.


```python
# 한글인지 체크
print(hgtk.checker.is_hangul('ㄱ'))
print(hgtk.checker.is_hangul('12'))
print(hgtk.checker.is_hangul('a'))
```

    True
    False
    False
    


```python
# 음절을 초성, 중성, 종성으로 분해
print(hgtk.letter.decompose('남'))
# 초성, 중성, 종성을 하나의 음절로 결합
print(hgtk.letter.compose('ㄴ', 'ㅏ', 'ㅁ'))
```

    ('ㄴ', 'ㅏ', 'ㅁ')
    남
    


```python
# 결합할 수 없는 상황에서는 에러 발생
try:
  hgtk.letter.compose('ㄴ', 'ㅁ', 'ㅁ') # 중성이 없는 경우
except:
  print('에러 발생')
```

    에러 발생
    

<br>

## 3. 데이터 전처리

fasttext는 subword 단위로 임베딩 벡터를 생성하는 도구이다. 한국어에서 subword는 자음 모음 단위로 생각할 수 있다. fasttext에 학습시킬 데이터를 만들기 위해 앞서 로드한 네이버 쇼핑 리뷰들을 hgtk를 활용해 자음 모음 단위로 전처리해줄 것이다.


```python
def word_to_jamo(token):
  def to_special_token(jamo): # 경우에 따라 초, 중, 종성이 다 있는 게 아닌 경우도 있다. 이 경우 -를 반환해주는 함수
    if not jamo:
      return '-'
    else:
      return jamo

  decomposed_token = ''
  for char in token:
    try:
      # char(음절)을 초성, 중성, 종성으로 분리
      cho, jung, jong = hgtk.letter.decompose(char)

      # 자모가 빈 문자일 경우 특수문자 -로 대체
      cho = to_special_token(cho)
      jung = to_special_token(jung)
      jong = to_special_token(jong)
      decomposed_token = decomposed_token + cho + jung + jong

    # 만약 char(음절)이 한글이 아닐 경우 자모를 나누지 않고 추가
    except Exception as exception:
      if type(exception).__name__ == 'NotHangulException':
        decomposed_token += char
    
  # 단어 토큰의 자모 단위 분리 결과를 추가
  return decomposed_token
```


```python
print(word_to_jamo('남동생'))
print(word_to_jamo('야구')) # 야구의 경우 종성이 없으므로 종성 부분을 -로 반환
```

    ㄴㅏㅁㄷㅗㅇㅅㅐㅇ
    ㅇㅑ-ㄱㅜ-
    


```python
print(mecab.morphs('선물용으로 빨리 받아서 전달했어야 하는 상품이었는데 머그컵만 와서 당황했습니다.'))
```

    ['선물', '용', '으로', '빨리', '받', '아서', '전달', '했어야', '하', '는', '상품', '이', '었', '는데', '머그', '컵', '만', '와서', '당황', '했', '습니다', '.']
    


```python
# mecab으로 형태소를 분리해주고 그 형태소마다 각각 자음모음을 분리해주는 함수
def tokenize_by_jamo(s):
    return [word_to_jamo(token) for token in mecab.morphs(s)]
```


```python
print(tokenize_by_jamo('선물용으로 빨리 받아서 전달했어야 하는 상품이었는데 머그컵만 와서 당황했습니다.'))
```

    ['ㅅㅓㄴㅁㅜㄹ', 'ㅇㅛㅇ', 'ㅇㅡ-ㄹㅗ-', 'ㅃㅏㄹㄹㅣ-', 'ㅂㅏㄷ', 'ㅇㅏ-ㅅㅓ-', 'ㅈㅓㄴㄷㅏㄹ', 'ㅎㅐㅆㅇㅓ-ㅇㅑ-', 'ㅎㅏ-', 'ㄴㅡㄴ', 'ㅅㅏㅇㅍㅜㅁ', 'ㅇㅣ-', 'ㅇㅓㅆ', 'ㄴㅡㄴㄷㅔ-', 'ㅁㅓ-ㄱㅡ-', 'ㅋㅓㅂ', 'ㅁㅏㄴ', 'ㅇㅘ-ㅅㅓ-', 'ㄷㅏㅇㅎㅘㅇ', 'ㅎㅐㅆ', 'ㅅㅡㅂㄴㅣ-ㄷㅏ-', '.']
    


```python
# 리뷰 데이터의 reviews 컬럼만을 가져와서 자모 분리
tokenized_data = []

for sample in tqdm(total_data['reviews'].to_numpy()):
    tokenzied_sample = tokenize_by_jamo(sample) # 자소 단위 토큰화
    tokenized_data.append(tokenzied_sample)
```

    100%|██████████| 200000/200000 [01:14<00:00, 2686.42it/s]
    


```python
print(len(tokenized_data))
print("전처리 전:", total_data['reviews'][1])
print("전처리 후:", tokenized_data[1])
```

    200000
    전처리 전: 택배가 엉망이네용 저희집 밑에층에 말도없이 놔두고가고
    전처리 후: ['ㅌㅐㄱㅂㅐ-', 'ㄱㅏ-', 'ㅇㅓㅇㅁㅏㅇ', 'ㅇㅣ-', 'ㄴㅔ-', 'ㅇㅛㅇ', 'ㅈㅓ-ㅎㅢ-', 'ㅈㅣㅂ', 'ㅁㅣㅌ', 'ㅇㅔ-', 'ㅊㅡㅇ', 'ㅇㅔ-', 'ㅁㅏㄹ', 'ㄷㅗ-', 'ㅇㅓㅄㅇㅣ-', 'ㄴㅘ-ㄷㅜ-', 'ㄱㅗ-', 'ㄱㅏ-', 'ㄱㅗ-']
    

단어를 자모 분리한 것을 역으로 하여 자모 상태를 단어로 다시 결합시키는 함수도 정의할 것이다. 이는 단어의 코사인 유사도를 평가할 때 자모 분리가 된 상태가 아니라 단어 상태로 편리하게 보기 위함이다.


```python
def jamo_to_word(jamo_sequence):
  tokenized_jamo = []
  index = 0
  
  # 1. 초기 입력
  # jamo_sequence = 'ㄴㅏㅁㄷㅗㅇㅅㅐㅇ'

  while index < len(jamo_sequence):
    # 문자가 한글(정상적인 자모)이 아닐 경우
    if not hgtk.checker.is_hangul(jamo_sequence[index]):
      tokenized_jamo.append(jamo_sequence[index])
      index = index + 1

    # 문자가 정상적인 자모라면 초성, 중성, 종성을 하나의 토큰으로 간주.
    else:
      tokenized_jamo.append(jamo_sequence[index:index + 3])
      index = index + 3

  # 2. 자모 단위 토큰화 완료
  # tokenized_jamo : ['ㄴㅏㅁ', 'ㄷㅗㅇ', 'ㅅㅐㅇ']
  
  word = ''
  try:
    for jamo in tokenized_jamo:

      # 초성, 중성, 종성의 묶음으로 추정되는 경우
      if len(jamo) == 3:
        if jamo[2] == "-":
          # 종성이 존재하지 않는 경우
          word = word + hgtk.letter.compose(jamo[0], jamo[1])
        else:
          # 종성이 존재하는 경우
          word = word + hgtk.letter.compose(jamo[0], jamo[1], jamo[2])
      # 한글이 아닌 경우
      else:
        word = word + jamo

  # 복원 중(hgtk.letter.compose) 에러 발생 시 초기 입력 리턴.
  # 복원이 불가능한 경우 예시) 'ㄴ!ㅁㄷㅗㅇㅅㅐㅇ'
  except Exception as exception:  
    if type(exception).__name__ == 'NotHangulException':
      return jamo_sequence

  # 3. 단어로 복원 완료
  # word : '남동생'

  return word
```

<br>

## 4. FastText


```python
import fasttext
```

fasttext를 실행하기에 앞서 훈련 대상인 단어들을 txt 파일로 준비해둬야 한다. 따라서 `tokenized_data.txt`라는 파일을 쓰기 모드(w)로 생성해주고 앞서 전처리한 `tokenized_data`를 입력해준다.


```python
with open('tokenized_data.txt', 'w') as out:
  for line in tqdm(tokenized_data, unit=' line'):
    out.write(' '.join(line) + '\n')
```

    100%|██████████| 200000/200000 [00:00<00:00, 286973.78 line/s]
    

아래처럼 훈련할 단어가 담긴 txt 파일을 지정하고 model을 `cbow`나 `skipgram` 중에 하나를 고르면 된다.


```python
model = fasttext.train_unsupervised('tokenized_data.txt', model='cbow')
```


```python
model.save_model("fasttext.bin")
```


```python
model = fasttext.load_model("fasttext.bin")
```


```python
model[word_to_jamo('남동생')] # 'ㄴㅏㅁㄷㅗㅇㅅㅐㅇ'
```




    array([-0.15177222, -0.74635   , -0.8032616 ,  0.19086838, -0.778221  ,
           -0.33223176, -0.11894439, -0.72866446,  0.37754712,  0.33814308,
            0.36616278, -0.49859792,  0.15212303, -0.787831  , -0.25276604,
           -0.10590696, -0.02901313, -0.19102712, -0.40464807,  0.37677866,
            0.37623113, -0.44016936, -0.6346251 , -0.4784998 ,  0.36894986,
            0.06783067, -0.20056958,  0.85232157,  0.9760077 , -0.20355019,
            0.46145713,  0.00457911,  0.66409916, -0.60430694, -0.45035866,
            0.5918075 , -1.1969922 , -0.2010963 , -0.36899805,  0.02050064,
           -0.14902137,  0.41190356, -0.09631964,  0.123348  , -0.19831082,
           -0.9359135 , -0.20674314, -0.48348927, -0.09120885,  0.1754866 ,
            0.4776521 ,  0.7623648 , -0.07341626,  0.18894948,  0.5302282 ,
            0.7917357 ,  0.48697317, -0.05473014,  0.34746835, -0.3974606 ,
           -0.86817527,  0.01599673,  1.035565  ,  0.56748825,  0.405412  ,
            0.35544434, -0.4864636 ,  0.05915926, -0.34068304,  0.39836073,
           -0.52921456, -0.3147612 , -0.2996531 , -0.7403795 ,  0.2946075 ,
            0.7669133 ,  0.6944267 , -0.2107564 , -0.48978466,  0.20623395,
            1.4869727 , -0.6125487 ,  0.5904486 ,  0.17088331,  0.34967554,
            0.17796417, -0.3819747 , -0.66418344,  0.41854003,  0.6213065 ,
           -0.8461816 , -0.30119175, -0.42944723,  0.9458985 ,  0.36967832,
            0.25508663, -0.76534146,  0.03767111,  1.6508778 , -0.7287428 ],
          dtype=float32)



'남동생'이라는 단어와 가장 유사도가 높은 단어들(자모 분리된 상태)을 k개만큼 출력해준다.


```python
model.get_nearest_neighbors(word_to_jamo('남동생'), k=10)
```




    [(0.8813450932502747, 'ㄷㅗㅇㅅㅐㅇ'),
     (0.8390687108039856, 'ㄴㅏㅁㅊㅣㄴ'),
     (0.7702850103378296, 'ㅅㅐㅇㅇㅣㄹ'),
     (0.752815306186676, 'ㄴㅏㅁㅇㅏ-'),
     (0.7487542629241943, 'ㅊㅣㄴㄱㅜ-'),
     (0.7453957200050354, 'ㄴㅏㅁㅍㅕㄴ'),
     (0.7136173248291016, 'ㅈㅗ-ㅋㅏ-'),
     (0.7125541567802429, 'ㅇㅓㄴㄴㅣ-'),
     (0.7080659866333008, 'ㄴㅏㄴㅅㅐㅇ'),
     (0.7055453658103943, 'ㄴㅏㅁㅈㅏ-')]



앞서 만든 `jamo_to_word`로 가독성이 좋게 출력한다.


```python
def transform(word_sequence):
  return [(jamo_to_word(word), similarity) for (similarity, word) in word_sequence]
```


```python
print(transform(model.get_nearest_neighbors(word_to_jamo('남동생'), k=10)))
print(transform(model.get_nearest_neighbors(word_to_jamo('구매'), k=10)))
print(transform(model.get_nearest_neighbors(word_to_jamo('배달'), k=10)))
```

    [('동생', 0.8813450932502747), ('남친', 0.8390687108039856), ('생일', 0.7702850103378296), ('남아', 0.752815306186676), ('친구', 0.7487542629241943), ('남편', 0.7453957200050354), ('조카', 0.7136173248291016), ('언니', 0.7125541567802429), ('난생', 0.7080659866333008), ('남자', 0.7055453658103943)]
    [('구매처', 0.8525390028953552), ('구입', 0.8198325037956238), ('주문', 0.7629261016845703), ('주문건', 0.6979227066040039), ('주문서', 0.6397473216056824), ('구매자', 0.6206300854682922), ('구메', 0.6068757772445679), ('구토', 0.6061024069786072), ('이용', 0.5840553641319275), ('재입', 0.5768207311630249)]
    [('배송지', 0.8438760042190552), ('깨달', 0.7850202322006226), ('매달', 0.7729867100715637), ('메달', 0.7509331107139587), ('백배', 0.7500661015510559), ('택배', 0.7392227053642273), ('송장', 0.7373930811882019), ('배공', 0.7373281121253967), ('공지', 0.7181926369667053), ('운송장', 0.7179238200187683)]
    

<br>

## 5. Word2Vec
이제 word2vec를 사용하여 자모 단위로 분리하는 것이 아닌 단어 단위로 분리하여 임베딩 벡터를 생성해볼 것이다.


```python
# 불용어 정의
stopwords = ['의','가','이','은','들','는','좀','잘','걍','과','도','를','으로','자','에','와','한','하다']

tokenized_data2 = []
for sentence in tqdm(total_data['reviews'].to_list()):
    tokenized_sentence = mecab.morphs(sentence) # 토큰화
    stopwords_removed_sentence = [word for word in tokenized_sentence if not word in stopwords] # 불용어 제거
    tokenized_data2.append(stopwords_removed_sentence)
```

    100%|██████████| 200000/200000 [01:09<00:00, 2871.90it/s]
    


```python
print("fasttext용 데이터:", tokenized_data2[0])
print("word2vec용 데이터:", tokenized_data[0])
```

    fasttext용 데이터: ['배공', '빠르', '고', '굿']
    word2vec용 데이터: ['ㅂㅐ-ㄱㅗㅇ', 'ㅃㅏ-ㄹㅡ-', 'ㄱㅗ-', 'ㄱㅜㅅ']
    


```python
from gensim.models import Word2Vec

model2 = Word2Vec(sentences = tokenized_data2, size = 1000, window = 5, min_count = 5, workers = 4, sg = 0)
```


```python
# 완성된 임베딩 매트릭스의 크기 확인
# 단어의 총 개수는 14959개이고 벡터 차원은 1000으로 축소되었다.
model2.wv.vectors.shape
```




    (14959, 1000)


<br>

## 6. FastText와 Word2Vec 결과 비교


```python
print("FastText 유사도:", transform(model.get_nearest_neighbors(word_to_jamo('남동생'), k=10)))
print("Word2Vec 유사도:", model2.wv.most_similar("남동생"))
```

    FastText 유사도: [('동생', 0.8813450932502747), ('남친', 0.8390687108039856), ('생일', 0.7702850103378296), ('남아', 0.752815306186676), ('친구', 0.7487542629241943), ('남편', 0.7453957200050354), ('조카', 0.7136173248291016), ('언니', 0.7125541567802429), ('난생', 0.7080659866333008), ('남자', 0.7055453658103943)]
    Word2Vec 유사도: [('친한', 0.7148764729499817), ('엄니', 0.7146899700164795), ('편찮', 0.7134093642234802), ('래미', 0.7106471061706543), ('어린이날', 0.6990622282028198), ('입학', 0.6911965608596802), ('친정', 0.6903277039527893), ('기념일', 0.6902635097503662), ('개업', 0.6821399927139282), ('사춘기', 0.6806015968322754)]
    


```python
print("FastText 유사도:", transform(model.get_nearest_neighbors(word_to_jamo('주문'), k=10)))
print("Word2Vec 유사도:", model2.wv.most_similar("주문"))
```

    FastText 유사도: [('주문건', 0.9059317708015442), ('주문서', 0.8423793315887451), ('구입', 0.7743308544158936), ('구매', 0.7629262208938599), ('주문자', 0.7511577606201172), ('주무시', 0.71355140209198), ('주무', 0.6975243091583252), ('구매처', 0.6918202638626099), ('시켰었', 0.6846689581871033), ('시킨', 0.6711220741271973)]
    Word2Vec 유사도: [('구매', 0.827530026435852), ('구입', 0.8101752996444702), ('선택', 0.634414792060852), ('결제', 0.5618018507957458), ('시켰', 0.5513455867767334), ('준비', 0.5279896259307861), ('시킨', 0.5216350555419922), ('도전', 0.5213436484336853), ('도착', 0.5197646617889404), ('신청', 0.5068678855895996)]
    

FastText로 계산된 유사도를 보면 Word2Vec에 비해 높은 유사도를 보여줌과 동시에 의미와 형태도 더욱 근접해있는 것을 체감할 수 있다.

또한 FastText가 조금 더 단어의 생김새에 주목하는 것처럼 체감된다. 예컨대 Word2Vec는 '주문'이라는 단어에 대해 '구입', '이용', '선택' 등 생김새에 차이가 있는 다어들도 유사한 것으로 출력한 반면, FastText는 '주문건', '주문서', '주무시' 등 생김새가 닮아있는 단어들을 우선적으로 출력하고 있다. 물론 대부분 의미적으로도 맞닿아있는 단어들을 잘 출력하고 있는 것으로 보인다.

해당 데이터셋만을 놓고보았을 때는 FastText의 성능이 보다 나은 것으로 보인다.
