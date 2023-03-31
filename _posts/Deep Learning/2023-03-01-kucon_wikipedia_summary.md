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

본 코드는 원하는 위키피디아 문서를 요약하는 시스템을 직접 구현한 것입니다. 사용한 모델은 huggingface의 KoBART 모델 3가지입니다.

<br>

## wikipedia api
wikipedia api는 위키피디아 문서들을 일일이 크롤링하지 않아도 쉽게 접근할 수 있게끔 하는 api로서 작동 방법 역시 간편하다.


```python
!pip install wikipedia-api
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting wikipedia-api
      Downloading Wikipedia_API-0.5.8-py3-none-any.whl (13 kB)
    Requirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from wikipedia-api) (2.25.1)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->wikipedia-api) (1.24.3)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->wikipedia-api) (2022.12.7)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->wikipedia-api) (2.10)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->wikipedia-api) (4.0.0)
    Installing collected packages: wikipedia-api
    Successfully installed wikipedia-api-0.5.8
    


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-1-d1c86ca9eaf7> in <module>
          1 get_ipython().system('pip install wikipedia-api')
    ----> 2 wiki = wikipediaapi.Wikipedia('ko') # change to korean version
    

    NameError: name 'wikipediaapi' is not defined



```python
!pip install konlpy
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting konlpy
      Downloading konlpy-0.6.0-py2.py3-none-any.whl (19.4 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m19.4/19.4 MB[0m [31m42.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: lxml>=4.1.0 in /usr/local/lib/python3.8/dist-packages (from konlpy) (4.9.2)
    Requirement already satisfied: numpy>=1.6 in /usr/local/lib/python3.8/dist-packages (from konlpy) (1.22.4)
    Collecting JPype1>=0.7.0
      Downloading JPype1-1.4.1-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (465 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m465.6/465.6 KB[0m [31m19.1 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: packaging in /usr/local/lib/python3.8/dist-packages (from JPype1>=0.7.0->konlpy) (23.0)
    Installing collected packages: JPype1, konlpy
    Successfully installed JPype1-1.4.1 konlpy-0.6.0
    


```python
!pip install pyLDAvis
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting pyLDAvis
      Downloading pyLDAvis-3.4.0-py3-none-any.whl (2.6 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m2.6/2.6 MB[0m [31m56.6 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: pandas>=1.3.4 in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (1.3.5)
    Requirement already satisfied: scipy in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (1.7.3)
    Requirement already satisfied: setuptools in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (57.4.0)
    Requirement already satisfied: gensim in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (3.6.0)
    Requirement already satisfied: jinja2 in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (2.11.3)
    Requirement already satisfied: numexpr in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (2.8.4)
    Requirement already satisfied: scikit-learn>=1.0.0 in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (1.0.2)
    Collecting funcy
      Downloading funcy-1.18-py2.py3-none-any.whl (33 kB)
    Requirement already satisfied: joblib>=1.2.0 in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (1.2.0)
    Requirement already satisfied: numpy>=1.22.0 in /usr/local/lib/python3.8/dist-packages (from pyLDAvis) (1.22.4)
    Requirement already satisfied: pytz>=2017.3 in /usr/local/lib/python3.8/dist-packages (from pandas>=1.3.4->pyLDAvis) (2022.7.1)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.8/dist-packages (from pandas>=1.3.4->pyLDAvis) (2.8.2)
    Requirement already satisfied: threadpoolctl>=2.0.0 in /usr/local/lib/python3.8/dist-packages (from scikit-learn>=1.0.0->pyLDAvis) (3.1.0)
    Requirement already satisfied: smart-open>=1.2.1 in /usr/local/lib/python3.8/dist-packages (from gensim->pyLDAvis) (6.3.0)
    Requirement already satisfied: six>=1.5.0 in /usr/local/lib/python3.8/dist-packages (from gensim->pyLDAvis) (1.15.0)
    Requirement already satisfied: MarkupSafe>=0.23 in /usr/local/lib/python3.8/dist-packages (from jinja2->pyLDAvis) (2.0.1)
    Installing collected packages: funcy, pyLDAvis
    Successfully installed funcy-1.18 pyLDAvis-3.4.0
    


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
import wikipediaapi
import pandas as pd
import re
from konlpy.tag import Okt
#okt = Okt()
#with open('/content/drive/MyDrive/sentence/stopwords.txt',  encoding='cp949') as f:
#    list_file = f.readlines()
#    stopwords = list_file[0].split(",")
from wordcloud import WordCloud
import matplotlib.pyplot as plt
import gensim
from gensim.models import LdaModel
import pyLDAvis.gensim
from gensim.models.coherencemodel import CoherenceModel
```


```python
wiki=wikipediaapi.Wikipedia('ko') # 한글 버전으로 변경
```

<br>

## transformers 모듈 설치 및 허깅페이스에서 모델 불러오기


```python
!pip install transformers
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting transformers
      Downloading transformers-4.26.1-py3-none-any.whl (6.3 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m6.3/6.3 MB[0m [31m55.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: requests in /usr/local/lib/python3.8/dist-packages (from transformers) (2.25.1)
    Requirement already satisfied: packaging>=20.0 in /usr/local/lib/python3.8/dist-packages (from transformers) (23.0)
    Requirement already satisfied: regex!=2019.12.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (2022.6.2)
    Requirement already satisfied: pyyaml>=5.1 in /usr/local/lib/python3.8/dist-packages (from transformers) (6.0)
    Collecting tokenizers!=0.11.3,<0.14,>=0.11.1
      Downloading tokenizers-0.13.2-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (7.6 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m7.6/7.6 MB[0m [31m48.8 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting huggingface-hub<1.0,>=0.11.0
      Downloading huggingface_hub-0.12.1-py3-none-any.whl (190 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m190.3/190.3 KB[0m [31m14.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: filelock in /usr/local/lib/python3.8/dist-packages (from transformers) (3.9.0)
    Requirement already satisfied: numpy>=1.17 in /usr/local/lib/python3.8/dist-packages (from transformers) (1.22.4)
    Requirement already satisfied: tqdm>=4.27 in /usr/local/lib/python3.8/dist-packages (from transformers) (4.64.1)
    Requirement already satisfied: typing-extensions>=3.7.4.3 in /usr/local/lib/python3.8/dist-packages (from huggingface-hub<1.0,>=0.11.0->transformers) (4.5.0)
    Requirement already satisfied: chardet<5,>=3.0.2 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (4.0.0)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (1.24.3)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2022.12.7)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.8/dist-packages (from requests->transformers) (2.10)
    Installing collected packages: tokenizers, huggingface-hub, transformers
    Successfully installed huggingface-hub-0.12.1 tokenizers-0.13.2 transformers-4.26.1
    


```python
import torch
from transformers import PreTrainedTokenizerFast
from transformers import BartForConditionalGeneration

digit_tokenizer = PreTrainedTokenizerFast.from_pretrained('digit82/kobart-summarization')
digit_model = BartForConditionalGeneration.from_pretrained('digit82/kobart-summarization')
```


    Downloading (…)okenizer_config.json:   0%|          | 0.00/295 [00:00<?, ?B/s]



    Downloading (…)/main/tokenizer.json:   0%|          | 0.00/682k [00:00<?, ?B/s]



    Downloading (…)cial_tokens_map.json:   0%|          | 0.00/109 [00:00<?, ?B/s]



    Downloading (…)lve/main/config.json:   0%|          | 0.00/1.20k [00:00<?, ?B/s]


    You passed along `num_labels=3` with an incompatible id to label map: {'0': 'NEGATIVE', '1': 'POSITIVE'}. The number of labels wil be overwritten to 2.
    


    Downloading (…)"pytorch_model.bin";:   0%|          | 0.00/496M [00:00<?, ?B/s]



```python
gogamza_tokenizer = PreTrainedTokenizerFast.from_pretrained('gogamza/kobart-summarization')
gogamza_model = BartForConditionalGeneration.from_pretrained('gogamza/kobart-summarization')
```


    Downloading (…)/main/tokenizer.json:   0%|          | 0.00/682k [00:00<?, ?B/s]



    Downloading (…)in/added_tokens.json:   0%|          | 0.00/4.00 [00:00<?, ?B/s]



    Downloading (…)cial_tokens_map.json:   0%|          | 0.00/111 [00:00<?, ?B/s]



    Downloading (…)lve/main/config.json:   0%|          | 0.00/1.18k [00:00<?, ?B/s]


    You passed along `num_labels=3` with an incompatible id to label map: {'0': 'NEGATIVE', '1': 'POSITIVE'}. The number of labels wil be overwritten to 2.
    The tokenizer class you load from this checkpoint is not the same type as the class this function is called from. It may result in unexpected tokenization. 
    The tokenizer class you load from this checkpoint is 'BartTokenizer'. 
    The class this function is called from is 'PreTrainedTokenizerFast'.
    You passed along `num_labels=3` with an incompatible id to label map: {'0': 'NEGATIVE', '1': 'POSITIVE'}. The number of labels wil be overwritten to 2.
    


    Downloading (…)"pytorch_model.bin";:   0%|          | 0.00/496M [00:00<?, ?B/s]



```python
ainize_tokenizer = PreTrainedTokenizerFast.from_pretrained('ainize/kobart-news')
ainize_model = BartForConditionalGeneration.from_pretrained('ainize/kobart-news')
```


    Downloading (…)okenizer_config.json:   0%|          | 0.00/302 [00:00<?, ?B/s]



    Downloading (…)/main/tokenizer.json:   0%|          | 0.00/682k [00:00<?, ?B/s]



    Downloading (…)cial_tokens_map.json:   0%|          | 0.00/239 [00:00<?, ?B/s]



    Downloading (…)lve/main/config.json:   0%|          | 0.00/1.45k [00:00<?, ?B/s]


    You passed along `num_labels=3` with an incompatible id to label map: {'0': 'NEGATIVE', '1': 'POSITIVE'}. The number of labels wil be overwritten to 2.
    


    Downloading (…)"pytorch_model.bin";:   0%|          | 0.00/496M [00:00<?, ?B/s]

<br>

## Kukipedia 소개


```python
class Kukipedia():

  def __init__(self, target_title):
    self.section_dict = {}
    self.target_title = target_title
    self.model_dict = {'digit': self.digit,
                  'gogamza': self.gogamza,
                  'ainize': self.ainize}


  def sections_dict(self, sections):
    """
    섹션들을 딕셔너리로 모아주는 함수
    """
    for s in sections:
      self.section_dict[s.title] = s.text
      self.sections_dict(s.sections)
    return self.section_dict


  def del_unnecessary_section(self, dictionary):
    """
    섹션 중에는 내용이 없는 것도 있고, 외부링크, 같이보기처럼 쓸모 없는 것도 있다. 
    그러한 섹션은 요약이 무의미하니 삭제하는 함수
    """
    no_contents = []
    reference_etc_list = ['각주', '같이 보기', '외부 링크'] # 요약의 효용이 없는 섹션
    for i in dictionary.keys():
      if not bool(dictionary[i]):
        no_contents.append(i)
      elif i in reference_etc_list:
        no_contents.append(i)
    for j in no_contents:
      del dictionary[j]
    return dictionary


  def make_wordcloud_entire(self, backgroundcolor='white', width=600, height=400):
    """
    문서 전체의 텍스트로 워드클라우드 만드는 함수
    요약문으로 워드클라우드 만드는 함수(make_wordcloud)보다 체감상 성능이 우수
    """
    target_page = wiki.page(self.target_title)
    entire_text = target_page.text
    corpus = self.preprocess_text(entire_text)
    wordcloud = WordCloud(font_path = '/content/drive/MyDrive/sentence/MALGUN.TTF', 
                          background_color = backgroundcolor, 
                          width = width, 
                          height = height).generate(corpus)
    plt.figure(figsize = (15 , 10))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.show()


  def get_url(self):
    """
    문서의 url을 출력해주는 함수
    """
    target_page = wiki.page(self.target_title)
    print("[{}] 문서의 url주소: {}".format(self.target_title, target_page.fullurl))


  def digit(self, text, num_beams=4, length_penalty=1.0, max_length=512):
    """
    digit82_KoBART 모델로 요약하는 함수
    Args:
      text (str): 요약 대상 원문 텍스트
      num_beams (int): beam search에 사용할 시퀀스 개수
      length_penalty (float): 요약문의 길이에 영향을 주는 가중치
      max_length (int): 요약문 토큰의 최대 길이
    Returns:
      result: 요약 결과물
    """
    raw_input_ids = digit_tokenizer.encode(text) # 토큰화, 정수 인코딩
    raw_input_ids = raw_input_ids[:1022] # KoBART 토큰 길이 제한 맞추기
    input_ids = [digit_tokenizer.bos_token_id] + raw_input_ids + [digit_tokenizer.eos_token_id] # bos, eos 토큰 추가
    summary_ids = digit_model.generate(torch.tensor([input_ids]), 
                                       num_beams=num_beams, 
                                       max_length=max_length, 
                                       length_penalty=length_penalty, 
                                       eos_token_id=1)
    result = digit_tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
    return result


  def gogamza(self, text, num_beams=4, length_penalty=1.0, max_length=512):
    """
    gogamza_KoBART 모델로 요약하는 함수
    """
    raw_input_ids = gogamza_tokenizer.encode(text) # 토큰화, 정수 인코딩
    raw_input_ids = raw_input_ids[:1022] # KoBART 토큰 길이 제한 맞추기
    input_ids = [gogamza_tokenizer.bos_token_id] + raw_input_ids + [gogamza_tokenizer.eos_token_id] # bos, eos 토큰 추가
    summary_ids = gogamza_model.generate(torch.tensor([input_ids]), 
                                         num_beams=num_beams, 
                                         max_length=max_length, 
                                         length_penalty=length_penalty, 
                                         eos_token_id=1)
    result = gogamza_tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
    return result


  def ainize(self, text, num_beams=4, length_penalty=1.0, max_length=512):
    """
    ainize_KoBART 모델로 요약하는 함수
    """
    input_ids = ainize_tokenizer.encode(text)
    input_ids = input_ids[1:len(input_ids)-1] # KoBART 토큰 길이 제한 맞추기
    input_ids = [ainize_tokenizer.bos_token_id] + input_ids[:1022] + [ainize_tokenizer.eos_token_id] # 토큰화, 정수 인코딩, torch 자료형으로 변경
    summary_ids = ainize_model.generate(torch.tensor([input_ids]), 
                                        num_beams=num_beams, 
                                        max_length=max_length, 
                                        length_penalty=length_penalty, 
                                        eos_token_id=1)
    result = ainize_tokenizer.decode(summary_ids.squeeze().tolist(), skip_special_tokens=True)
    return result


  def wiki_summary_outline(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512):
    """
    문서의 개요 부분을 요약하는 함수
    """
    target_page = wiki.page(self.target_title)
    if target_page.exists():
      model = self.model_dict[summary_model]
      text = target_page.summary
      result = model(text, num_beams, length_penalty, max_length)

    else:
      result = "[{}](이)라는 이름의 문서는 위키피디아에 존재하지 않습니다".format(self.target_title)

    return result


  def wiki_summary_section(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512):
    """
    원하는 섹션 하나 골라 요약해주는 함수
    """
    model = self.model_dict[summary_model]
    target_page = wiki.page(self.target_title)

    if target_page.exists(): # 문서가 존재할 경우만 요약
      section_collection = self.sections_dict(target_page.sections)
      section_collection = self.del_unnecessary_section(section_collection)

      print(section_collection.keys())
      target_key = input('위의 keys 중 요약을 원하는 섹션을 따옴표 없이 정확히 입력해주세요 : ')
      if target_key in section_collection.keys():
        text = section_collection[target_key]
        result = model(text, num_beams, length_penalty, max_length)
      else:
        result = "[{}](이)라는 이름의 섹션은 [{}] 문서에 존재하지 않습니다.".format(target_key, self.target_title)

    else: # 문서가 존재하지 않으면 아래 문구 출력
      result = "[{}](이)라는 이름의 문서는 위키피디아에 존재하지 않습니다".format(self.target_title)

    return result


  def wiki_summary_combine(self, summary_model, num_beams=4, length_penalty=1.0, 
                           max_length=512, return_dict=False):
    """
    세션을 각각 요약한 것을 하나로 합쳐주는 함수
    """
    model = self.model_dict[summary_model]
    target_page = wiki.page(self.target_title)

    if target_page.exists(): # 문서가 존재할 경우만 요약
      section_collection = self.sections_dict(target_page.sections)
      section_collection = self.del_unnecessary_section(section_collection)
      summary_dict = {}
      for k in section_collection.keys():
        text = section_collection[k]
        summary = model(text, num_beams, length_penalty, max_length)
        summary_dict[k] = summary
      summary_dict_collect = summary_dict

      if return_dict: # return_dict=True인 경우 딕셔너리 형태로 반환
        result = summary_dict_collect

      else: # 개행된 문자열로 출력
        result = ""
        for i, j in enumerate(summary_dict_collect.keys()):
          result += "# section [{}]: [{}] \n    - summary: {} \n\n".format(i, j, 
                                                                           summary_dict_collect[j])

    else: # 문서가 존재하지 않으면 아래 문구 출력
      result = "[{}](이)라는 이름의 문서는 위키피디아에 존재하지 않습니다".format(self.target_title)

    return result


  def preprocess_text(self, text):
    """
    워드클라우드, 토픽모델링을 위해 전처리해주는 함수
    """
    okt = Okt()
    with open('/content/drive/MyDrive/sentence/stopwords.txt',  encoding='cp949') as f:
      list_file = f.readlines()
      stopwords = list_file[0].split(",")

    only_korean = re.sub('[^가-힣]', ' ', text)
    spaced_text = ' '.join(only_korean.split())
    morph_text = okt.morphs(spaced_text, stem=True)
    lenght_two = [x for x in morph_text if len(x)>1]
    nonstopwords = [x for x in lenght_two if x not in stopwords]
    corpus = ' '.join(nonstopwords)
    return corpus

  def make_wordcloud(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512, 
                     backgroundcolor='white', width=600, height=400):
    """
    요약문을 토대로 워드클라우드를 만드는 함수
    """
    summary_dict = self.wiki_summary_combine(summary_model=summary_model,
                                     num_beams=num_beams,
                                     length_penalty=length_penalty,
                                     max_length=max_length,
                                     return_dict=True)
    entire_text = ' '.join(summary_dict.values())
    corpus = self.preprocess_text(entire_text)
    wordcloud = WordCloud(font_path = '/content/drive/MyDrive/sentence/MALGUN.TTF', 
                          background_color = backgroundcolor, 
                          width = width, 
                          height = height).generate(corpus)
    plt.figure(figsize = (15 , 10))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.show()


  def compute_coherence_values(self, dictionary, corpus, texts, limit=15, start=2, step=3):
    """
    토픽모델링을 하기 앞서 최적의 토픽 개수를 찾아주는 함수
    """
    coherence_values = {}
    for num_topics in range(start, limit, step):
        model = LdaModel(corpus=corpus, id2word=dictionary, num_topics=num_topics)
        coherencemodel = CoherenceModel(model=model, texts=texts, dictionary=dictionary, coherence='c_v')
        coherence_values[num_topics] = coherencemodel.get_coherence()

    return max(coherence_values, key=coherence_values.get)


  def topic_modeling(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512):
      """
      요약문을 토대로 토픽 모델링을 진행하는 함수
      """
      summary_dict = self.wiki_summary_combine(summary_model=summary_model,
                                     num_beams=num_beams,
                                     length_penalty=length_penalty,
                                     max_length=max_length,
                                     return_dict=True)
      preprocessed = []
      for text in summary_dict.values():
        preprocessed.append(self.preprocess_text(text).split())
      dic = gensim.corpora.Dictionary(preprocessed)
      bow_corpus = [dic.doc2bow(doc) for doc in preprocessed]
      NUM_TOPICS = self.compute_coherence_values(dic, bow_corpus, preprocessed)
      lda_model =  gensim.models.LdaModel(bow_corpus, 
                                num_topics = NUM_TOPICS, 
                                id2word = dic,                                    
                                passes = 10)
      pyLDAvis.enable_notebook()
      vis = pyLDAvis.gensim.prepare(lda_model, bow_corpus, dic)
      return vis

  def topic_modeling_entire(self):
    """
    문서 전체를 토대로 토픽 모델링을 진행하는 함수
    """
    target_page = wiki.page(self.target_title)

    if target_page.exists(): # 문서가 존재할 경우만 요약
      section_collection = self.sections_dict(target_page.sections)
      section_collection = self.del_unnecessary_section(section_collection)
      preprocessed = []
      for text in section_collection.values():
        preprocessed.append(self.preprocess_text(text).split())
      dic = gensim.corpora.Dictionary(preprocessed)
      bow_corpus = [dic.doc2bow(doc) for doc in preprocessed]
      NUM_TOPICS = self.compute_coherence_values(dic, bow_corpus, preprocessed)
      lda_model =  gensim.models.LdaModel(bow_corpus, 
                                num_topics = NUM_TOPICS, 
                                id2word = dic,                                    
                                passes = 10)
      pyLDAvis.enable_notebook()
      result = pyLDAvis.gensim.prepare(lda_model, bow_corpus, dic)

    else: # 문서가 존재하지 않으면 아래 문구 출력
      result = "[{}](이)라는 이름의 문서는 위키피디아에 존재하지 않습니다".format(self.target_title)
      
    return result

```


```python
help(Kukipedia)
```

    Help on class Kukipedia in module __main__:
    
    class Kukipedia(builtins.object)
     |  Kukipedia(target_title)
     |  
     |  Methods defined here:
     |  
     |  __init__(self, target_title)
     |      Initialize self.  See help(type(self)) for accurate signature.
     |  
     |  ainize(self, text, num_beams=4, length_penalty=1.0, max_length=512)
     |      ainize_KoBART 모델로 요약하는 함수
     |  
     |  compute_coherence_values(self, dictionary, corpus, texts, limit=15, start=2, step=3)
     |      토픽모델링을 하기 앞서 최적의 토픽 개수를 찾아주는 함수
     |  
     |  del_unnecessary_section(self, dictionary)
     |      섹션 중에는 내용이 없는 것도 있고, 외부링크, 같이보기처럼 쓸모 없는 것도 있다. 
     |      그러한 섹션은 요약이 무의미하니 삭제하는 함수
     |  
     |  digit(self, text, num_beams=4, length_penalty=1.0, max_length=512)
     |      digit82_KoBART 모델로 요약하는 함수
     |  
     |  get_url(self)
     |      문서의 url을 출력해주는 함수
     |  
     |  gogamza(self, text, num_beams=4, length_penalty=1.0, max_length=512)
     |      gogamza_KoBART 모델로 요약하는 함수
     |  
     |  make_wordcloud(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512, backgroundcolor='white', width=600, height=400)
     |      요약문을 토대로 워드클라우드를 만드는 함수
     |  
     |  make_wordcloud_entire(self, backgroundcolor='white', width=600, height=400)
     |      문서 전체의 텍스트로 워드클라우드 만드는 함수
     |      요약문으로 워드클라우드 만드는 함수(make_wordcloud)보다 체감상 성능이 우수
     |  
     |  preprocess_text(self, text)
     |      워드클라우드, 토픽모델링을 위해 전처리해주는 함수
     |  
     |  sections_dict(self, sections)
     |      섹션들을 딕셔너리로 모아주는 함수
     |  
     |  topic_modeling(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512)
     |      요약문을 토대로 토픽 모델링을 진행하는 함수
     |  
     |  wiki_summary_combine(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512, return_dict=False)
     |      세션을 각각 요약한 것을 하나로 합쳐주는 함수
     |  
     |  wiki_summary_outline(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512)
     |      문서의 개요 부분을 요약하는 함수
     |  
     |  wiki_summary_section(self, summary_model, num_beams=4, length_penalty=1.0, max_length=512)
     |      원하는 섹션 하나 골라 요약해주는 함수
     |  
     |  ----------------------------------------------------------------------
     |  Data descriptors defined here:
     |  
     |  __dict__
     |      dictionary for instance variables (if defined)
     |  
     |  __weakref__
     |      list of weak references to the object (if defined)
    
    
<br>

### 1. 요약을 원하는 문서명 입력


```python
kuki = Kukipedia(target_title = "고려대학교")
```


```python
# 존재하지 않는 문서명 입력 시에는
kuki = Kukipedia(target_title = "이런이름의문서는없다")
kuki.wiki_summary_outline('ainize')
```




    '[이런이름의문서는없다](이)라는 이름의 문서는 위키피디아에 존재하지 않습니다'

<br>

### 2. get_url()
입력한 문서의 url을 출력해주는 함수이다.


```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.get_url()
```

    [고려대학교] 문서의 url주소: https://ko.wikipedia.org/wiki/%EA%B3%A0%EB%A0%A4%EB%8C%80%ED%95%99%EA%B5%90
    

<br>

### 3. wiki_summary_outline(summary_model)

문서 최상단에 있는 '개요' 부분을 요약해주는 함수이다.

- `summary_model`: `gogamza`, `digit`, `ainize` 모델 중 택 1
- `num_beams`: n개의 확률 값이 높은 단어를 선택하기 위한 파라미터(default=4)
- `length_penalty`: 1보다 작은 경우 짧은 문장 생성 유도, 1보다 클 경우 더 긴 문장 생성 유도(default=1.0)
- `max_length`: 요약문의 최대 길이(default=512)


```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_outline(summary_model='ainize',
                          num_beams=4,
                          length_penalty=1.2,
                          max_length=512)
```




    '독제 정권에 항거하여 1960년 4·19 혁명의 촉매제가 된 4·18 의거를 비롯한 각종 시위의 중심에 서기도 했던 고려대학교는 1946년 종합대학으로 승격하여 고려대학교로 개칭하였으며, 1952년 12월에는 문과대학을 문리과대학으로 개편하고 1971년에는 우석대학교 의과대학을 흡수 합병해 이공·인문·의예·예체능 관련 학과를 고루 갖추게 됐다.'




```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_outline('gogamza')
```




    '대한민국의 사립 종합대학인 고려대학교는 1960년 4·19 혁명의 촉매제가 된 4·18 의거를 비롯한 각종 시위의 중심에 서기도 했다.'




```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_outline(summary_model='digit',
                          num_beams=4,
                          length_penalty=1.2,
                          max_length=512)
```




    '대한민국의 사립 종합대학인 고려대학교(高麗大學校, 영어: Korea University, KU)는 한국 최초의 민간인에 의한 근대 고등교육기관 보성전문학교(普成專門學校)로 출발하였으며 1960년 4·19 혁명의 촉매제가 된 4·18 의거를 비롯한 각종 시위의 중심에 서기도 하였다.'




```python
kuki = Kukipedia(target_title = "손흥민")
kuki.wiki_summary_outline('gogamza')
```




    '대한민국 축구 국가대표팀의 주장이자 2018년 아시안 게임 금메달리스트인 손흥민은 현재 잉글랜드 프리미어리그 토트넘 홋스퍼에서 윙어로 활약하고 있으며 2018년 아시안 게임 금메달리스트이며 영국에서는 애칭인 "쏘니"(Sonny)로 불린다.'


<br>

### 4. wiki_summary_section(summary_model)

문서에는 여러 가지 하위 섹션이 있다. 이 섹션 중 원하는 섹션을 입력하면 그 섹션만 요약해주는 함수이다.

- `summary_model`: `gogamza`, `digit`, `ainize` 모델 중 택 1
- `num_beams`: n개의 확률 값이 높은 단어를 선택하기 위한 파라미터(default=4)
- `length_penalty`: 1보다 작은 경우 짧은 문장 생성 유도, 1보다 클 경우 더 긴 문장 생성 유도(default=1.0)
- `max_length`: 요약문의 최대 길이(default=512)


```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_section('digit')
```

    dict_keys(['고려대학교의 모태', '종합대학으로의 승격과 우석대학교 의과대학의 흡수·합병', '반복적인 시위와 잦은 휴교 사태', '교육시설의 확충', '졸업요건의 변화', '개교 100주년 기념사업', '세계 대학 순위 평가', '세계 대학 평가 기관', '최근의 세계 대학 평가 결과', '개설 학과·전공', '학부 과정', '대학원 과정', '국제 교류', 'Global KU 프로젝트와 한자졸업요건', '제2 전공의 의무화', '캠퍼스 및 학교 소유 시설', '인문·사회계 캠퍼스', '자연계(이공계) 캠퍼스', '녹지 캠퍼스', '정릉캠퍼스', '세종캠퍼스(분교)', '의료원 및 기타 시설', '서울캠퍼스 주변 상권', '총학생회', '학내 공식 언론단체', '동아리', '기타', '사발식', '동아리 박람회', '4·18 기념 마라톤과 구국 대장정', '석탑대동제', '입실렌티 지·야의 함성', '고연전', '운동부', '교우회 및 동문'])
    위의 keys 중 요약을 원하는 섹션을 따옴표 없이 정확히 입력해주세요 : 반복적인 시위와 잦은 휴교 사태
    




    '1960년 4월 11일 3·15 부정선거와 관련된 마산 시위에서 실종된 김주열의 시신이 마산 앞바다에서 발견된 것과 관련하여 5개 단과대학 운영위원장들의 주도로 수많은 학생들이 ‘마산사건의 책임자를 즉시 처단’할 것을 요구하면서 장외시위를 벌였으나, 돌아오는 길에 신도환의 대한반공청년단 소속 폭력배들에게 피습을 당하여 4·19 혁명의 도화선이 됐다.'




```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_section('ainize',
                          num_beams=5)
```

    dict_keys(['고려대학교의 모태', '종합대학으로의 승격과 우석대학교 의과대학의 흡수·합병', '반복적인 시위와 잦은 휴교 사태', '교육시설의 확충', '졸업요건의 변화', '개교 100주년 기념사업', '세계 대학 순위 평가', '세계 대학 평가 기관', '최근의 세계 대학 평가 결과', '개설 학과·전공', '학부 과정', '대학원 과정', '국제 교류', 'Global KU 프로젝트와 한자졸업요건', '제2 전공의 의무화', '캠퍼스 및 학교 소유 시설', '인문·사회계 캠퍼스', '자연계(이공계) 캠퍼스', '녹지 캠퍼스', '정릉캠퍼스', '세종캠퍼스(분교)', '의료원 및 기타 시설', '서울캠퍼스 주변 상권', '총학생회', '학내 공식 언론단체', '동아리', '기타', '사발식', '동아리 박람회', '4·18 기념 마라톤과 구국 대장정', '석탑대동제', '입실렌티 지·야의 함성', '고연전', '운동부', '교우회 및 동문'])
    위의 keys 중 요약을 원하는 섹션을 따옴표 없이 정확히 입력해주세요 : 고연전
    




    '고연전은 연희전문학교와 연희학교의 전신인 연희전문학교가 1925년 5월 30일에 열린 조선체육회 주최 ‘제5회 전조선(全朝鮮) 정구대회’에서 처음으로 맞붙어 애교심을 고양하고 고려대학교와 연세대학교 양교의 친선을 도모하기 위한 목적으로 스포츠 경기를 비롯한 강연, 온라인 게임, 사회공헌활동과 같은 다양한 분야에서 연세대학교와 승부를 겨루는 행사 일체를 의미하지만 정식 명칭은 실질적으로 방송 중계나 신문 보도 등의 공식 석상에만 사용될 뿐이며, 고려대학교에서는 고연전, 연세대학교에서는 연고전이라는 명칭으로 통용된다.'




```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_section('digit')
```

    dict_keys(['고려대학교의 모태', '종합대학으로의 승격과 우석대학교 의과대학의 흡수·합병', '반복적인 시위와 잦은 휴교 사태', '교육시설의 확충', '졸업요건의 변화', '개교 100주년 기념사업', '세계 대학 순위 평가', '세계 대학 평가 기관', '최근의 세계 대학 평가 결과', '개설 학과·전공', '학부 과정', '대학원 과정', '국제 교류', 'Global KU 프로젝트와 한자졸업요건', '제2 전공의 의무화', '캠퍼스 및 학교 소유 시설', '인문·사회계 캠퍼스', '자연계(이공계) 캠퍼스', '녹지 캠퍼스', '정릉캠퍼스', '세종캠퍼스(분교)', '의료원 및 기타 시설', '서울캠퍼스 주변 상권', '총학생회', '학내 공식 언론단체', '동아리', '기타', '사발식', '동아리 박람회', '4·18 기념 마라톤과 구국 대장정', '석탑대동제', '입실렌티 지·야의 함성', '고연전', '운동부', '교우회 및 동문'])
    위의 keys 중 요약을 원하는 섹션을 따옴표 없이 정확히 입력해주세요 : 고연전
    




    '애교심을 고양하고 고려대학교와 연세대학교 양교의 친선을 도모하기 위해 매년 9월에 연세대학교 5개 종목의 운동부 선수들과 맞붙는 ‘정기 고연전’ 과 정기 고연전 이외의 기간에 열리는 ‘비정기 고연전’으로 나누어볼 수 있다.'




```python
kuki = Kukipedia(target_title = "손흥민")
kuki.wiki_summary_section('ainize')
```

    dict_keys(['학력', '초기 생애', '2010-11 시즌', '2011-12 시즌', '2013-14 시즌', '2014-15 시즌', '2015-16 시즌', '2016-17 시즌', '2017-18 시즌', '2018-19 시즌', '2019-20 시즌', '2020-21 시즌', '2021-22 시즌', '2022-23 시즌', '국가대표팀 경력', '주요 출전 국제대회', '대회 기록', '개인', '대한민국 정부', '클럽', '국가대표팀 주요 국제대회 참가 기록', '국가대표팀 득점 기록', '광고', '평판', '기타'])
    위의 keys 중 요약을 원하는 섹션을 따옴표 없이 정확히 입력해주세요 : 초기 생애
    




    '함92년 7월 8일 손흥민은 강원도 춘천시 후평동에서 아버지 손웅정과 어머니 길은자의 차남으로 태어나 그곳에서 자라 막노동을 했다고 할 정도로 집안 환경이 좋지 않아 학업에 열중은 못했으나 축구선수였던 아버지의 영향으로 유전적으로 축구에 재능이 있어 매일 10시간 이상을 땀을 흘려 축구 연습에만 매진했다고 회고했다.'




```python
# 존재하지 않는 섹션명을 입력할 경우
kuki = Kukipedia(target_title = "고려대학교")
kuki.wiki_summary_section('digit')
```

    dict_keys(['고려대학교의 모태', '종합대학으로의 승격과 우석대학교 의과대학의 흡수·합병', '반복적인 시위와 잦은 휴교 사태', '교육시설의 확충', '졸업요건의 변화', '개교 100주년 기념사업', '세계 대학 순위 평가', '세계 대학 평가 기관', '최근의 세계 대학 평가 결과', '개설 학과·전공', '학부 과정', '대학원 과정', '국제 교류', 'Global KU 프로젝트와 한자졸업요건', '제2 전공의 의무화', '캠퍼스 및 학교 소유 시설', '인문·사회계 캠퍼스', '자연계(이공계) 캠퍼스', '녹지 캠퍼스', '정릉캠퍼스', '세종캠퍼스(분교)', '의료원 및 기타 시설', '서울캠퍼스 주변 상권', '총학생회', '학내 공식 언론단체', '동아리', '기타', '사발식', '동아리 박람회', '4·18 기념 마라톤과 구국 대장정', '석탑대동제', '입실렌티 지·야의 함성', '고연전', '운동부', '교우회 및 동문'])
    위의 keys 중 요약을 원하는 섹션을 따옴표 없이 정확히 입력해주세요 : 알 수 없는 섹션
    




    '[알 수 없는 섹션](이)라는 이름의 섹션은 [고려대학교] 문서에 존재하지 않습니다.'


<br>

### 5. wiki_summary_combine(summary_model)

각 섹션을 일일이 요약한 후 요약문을 하나로 합쳐서 일괄적으로 보여주는 함수이다.

- `summary_model`: `gogamza`, `digit`, `ainize` 모델 중 택 1
- `num_beams`: n개의 확률 값이 높은 단어를 선택하기 위한 파라미터(default=4)
- `length_penalty`: 1보다 작은 경우 짧은 문장 생성 유도, 1보다 클 경우 더 긴 문장 생성 유도(default=1.0)
- `max_length`: 요약문의 최대 길이(default=512)

- `return_dict`를 `True`로 설정하면 딕셔너리 형태로 반환. `False`인 경우엔 개행된 문자열 형태로 출력(default=False)


```python
kuki = Kukipedia(target_title = "고려대학교")
wiki_summary_combine = kuki.wiki_summary_combine('ainize')
print(wiki_summary_combine)
```

    # section [0]: [고려대학교의 모태] 
        - summary: 고려대학교의 모태는 대한제국 내장원경 이용익이 1905년에 대한제국 내장원경 이용익이 1905년에 대한제국 내장원경 이용익이 1905년에 설립한 보성전문학교로, 1932년 동아일보 창업자 김성수가 학교를 인수했고 1934년에는 안암동의 현재의 자리로 학교를 옮겼다가 1944년 일제에 의해 학교 이름이 경성척식경제전문학교로 변경됐다. 
    
    # section [1]: [종합대학으로의 승격과 우석대학교 의과대학의 흡수·합병] 
        - summary: 1945년 해방 후 1945년 해방 후 교명을 보성전문학교로 환원하고, 1946년에 고려대학교로 개칭됐으며 1946년에 종합대학으로의 설립이 인가되고, 1946년에 본교는 애기능 인근 부지를 매입하였으며 1946년에 이공계캠퍼스가 자리 잡는 기반이 된다. 
    
    # section [2]: [반복적인 시위와 잦은 휴교 사태] 
        - summary: 독60년 4월 11일 3·15 부정선거와 관련된 마산 시위에서 실종된 김주열의 시신이 마산 앞바다에서 발견된 것과 관련하여 수많은 학생들이 ‘마산사건의 책임자를 즉시 처단’할 것을 요구하면서 장외시위를 벌였으나, 돌아오는 길에 신도환의 대한반공청년단 소속 폭력배들에게 피습을 당하여 4·19 혁명의 도화선이 되고 독재정권에 맞선 선배들의 정의로운 행동을 계승하기 위해 ‘4·18 구국대장정’ 행사가 매년 4월 18일에 열린다. 
    
    # section [3]: [교육시설의 확충] 
        - summary: 1945년 신관을 신축하여 1978년 3월에 개관한 중앙도서관 구관이 1935년에 착공하여 1937년에 개관한 데 이어 1975년에는 창립 70주년을 기념하기 위해 중앙도서관 신관을 신축하여 1978년 3월에 개관하였으며 한편 수도권 인구분산 정책에 따라 조치원분교 설립이 인가됐는데 이것이 현재의 세종캠퍼스가 탄생하는 밑거름이 됐다. 
    
    # section [4]: [졸업요건의 변화] 
        - summary: 2000년대에 들어서자 세계화 되는 추세에서 고려대학교 졸업자의 실력은 세계적인 수준에 맞추어야 한다는 ‘Global KU 프로젝트’ 등의 영향으로 졸업요건이 대폭 강화되는 등의 변화가 있었고, 2004년 3월 신입생부터는 한자 2급 수준의 졸업요건이 신설됐으나 2011년 들어 일부 학과들이 이 요건을 폐지했으며 2004년 3월 신입생부터는 심화전공이나 이중전공, 연계전공, 이렇게 세 가지 항목 가운데 한 가지를 의무적으로 선택하여 학생설계전공으로 세분화할 수 있는 제2 전공을 의무적으로 이수해야 졸업할 수 있게 됐다. 
    
    # section [5]: [개교 100주년 기념사업] 
        - summary: 2005년에는 고려대학교가 개교한 지 100주년이 되는 해로 같은 해 5월 5일에는 개교 100주년 기념식이 거행됐고, 2003년에 ‘개교 100주년 기념사업회’가 출범 및 지층에 7천평 규모의 1백주년 기념관에는 박물관과 디지털 도서관을 건립 하는 등 교내에서 다양한 기념사업들이 추진됐다. 
    
    # section [6]: [세계 대학 순위 평가] 
        - summary: 고윤대 총장의 재임기간 동안 고려대학교는 2006년 영국 《타임즈》지가 발표한 세계대학순위에서 종합 150위에 오르는 등 많은 성과를 기록하였으며, 2009년에는 경영대학 평가 전문기관인 에듀니버설이 전 세계 경영대 학장 1000명을 상대로 한 설문조사에서 ‘추천하고 싶은 대학’ 국내 1위에 올랐으며, 2010년 대한민국 경영대 평가 설문 조사에서 1위를 차지한 데 이어 2012년 UTD 랭킹에서는 86위에 올라 대한민국 대학 중에서 1위를 차지하였고, BK21의 후속 사업인 BK21 플러스의 2013년 초기 선정 결과도 사업단 수와 지원 금액에서 서울대학교에 이어 2위를 차지하였으며, 특히 공과대학의 모든 분야가 BK21 플러스에 선정됐다. 
    
    # section [7]: [세계 대학 평가 기관] 
        - summary: 고려대학교는 세계에서 가장 유명하고 영향력 있는 3대 세계 대학 평가 기관인 QS가 발표한 세계대학평가 학과별 평가에서 전체 평가 항목 36개 분야 중 20개 학과에서 세계 100위권 이내에 드는 우수한 성과를 보였고, 미국에서 가장 유명한 대학 평가 기관인 US 뉴스&월드 리포트에서는 157위로 대한민국 대학들 중에서는 서울대학교의 뒤를 이어서 2위를 차지했다. 
    
    # section [8]: [최근의 세계 대학 평가 결과] 
        - summary: 2016년 9월 발표한 QS 세계대학평가에서는 전년 대비 6순위 상승한 98위를 기록하며 대한민국 종합사립대학으로서는 가장 먼저 세계 100위 안에 진입한 대학이 됐고,세계 대학순위 100위권에는 고려대학교 외 세 곳이 포함되었으며 국립 대학인 서울대학교, 한국과학기술원, 포항공과대학교 등이 포함되었으며 2018년에는 전년 대비 4순위 향상한 86위에 등극했다. 
    
    # section [9]: [개설 학과·전공] 
        - summary: 전부와 대학원 과정이 모두 개설되어 있고 과정이 모두 개설되어 있는 가운데 학부와 대학원 과정이 모두 개설되어 있는 중이다. 
    
    # section [10]: [학부 과정] 
        - summary: 2022년 기준으로 본교인 서울캠퍼스에는 12개 단과대학, 분교인 세종캠퍼스에는 5개 단과대학이 설치돼 있으며 2009학년도부터 법학전문대학원(로스쿨)이 개원함에 따라 법과대학은 폐지되었으며, 대신 자유전공학부를 신설하여 기존의 법학과 신입생 정원을 대체하고 있고 컴퓨터교육과의 경우 2014년학년도를 시작으로 컴퓨터학과로 개편되면서 더이상 사범대학이 아닌 정보통신대학의 소속으로 변경됐다. 
    
    # section [11]: [대학원 과정] 
        - summary: 의학대학원은 일반대학원과 전문대학원, 특수대학원로 구분되어 있으며, 의학전문대학원의 경우 전문대학원 53명을 의학과 53명과 병행하여 임시로 운영하였으나 2015년에 폐지 되었고, 2015년에 폐지 되었고, 의학전문대학원의 경우 전문대학원 53명을 의학과 53명과 병행하여 임시로 운영하였으나 2015년에 폐지되었다. 
    
    # section [12]: [국제 교류] 
        - summary: 국제처에서는 국제 교류의 갈래로 SEP(Student Exchange Program)와 VSP(Visiting Student Program)의 두 종류를 운영하고 있는데, SEP는 본교와 학술 교류 협정이 체결된 외국 대학과 학부 또는 대학원생을 교환하는 프로그램이고, VSP는 소수의 학교와 별도의 협정을 체결하여 본교의 학생들을 대규모로 파견하는 것이 VSP 프로그램의 요지이다. 
    
    # section [13]: [Global KU 프로젝트와 한자졸업요건] 
        - summary: 세계 100대 대학에 진입을 목표로 하는 Global KU 프로젝트는 장기적으로 영어강의 확충, 해외 대학과의 교류 증진, 외국인 교수 채용 확대 등을 목표로 하고 있으며, 졸업요건으로 요구하는 토익 점수를 대폭 상승시키고, 2005년 5월에는 개교 100주년을 기념하여 세계대학총장포럼을 개최 하는 등 단기적인 계획도 함께 진행하고 있으며 교내에 도입된 영어강의와 관련해서 학교 측에서는 각 단과대의 특성을 고려해 앞으로 영어로 전공 수업등이 진행되는 영어강의와 관련된 졸업 요건을 단과대별로 완화하거나 강화할 계획인 것으로 알려져 있다. 
    
    # section [14]: [제2 전공의 의무화] 
        - summary: 제전공 이수학점이 상대적으로 연세대학교보다 10 학점이상 낮기 때문에 연세대학교에서 시행되는 전과 제도를 이용하지 않고도 복수로 전공이 가능하기 때문에 학교 관계자학적·수업지원팀 유신열 과장이 제1전공을 이수하면서 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공과 별도로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 학점은 21학점까지 인정받을 수 있다고 말했다. 
    
    # section [15]: [캠퍼스 및 학교 소유 시설] 
        - summary: 본캠퍼스는 인문·사회계 캠퍼스, 자연계지역, 녹지 캠퍼스, 정릉캠퍼스를 포함한 가운데 어윤대 총장이 재임하던 시기에 3,500억 원의 발전 기금을 유치하고 기업체의 후원을 받아내 교내 전체 건물의 40% 정도를 신축 및 리모델링함에 따라 전체적으로 건물 양식이 일관된 형태를 보이고 있고 어윤대 총장이 재임하던 시기에 3,500억 원의 발전 기금을 유치하고 기업체의 후원을 받아내 교내 전체 건물의 40% 정도를 신축 및 리모델링함에 따라 전체적으로 건물 양식이 일관된 형태를 보이고 있으며 중앙광장과 하나스퀘어가 각각 인문사회계지역과 자연계지역에 위치하여 각 지역의 중심을 이루고 있다. 
    
    # section [16]: [인문·사회계 캠퍼스] 
        - summary: 인광장과 본관이 위치한 지역으로, 본관 ~ 중앙광장 ~ 정문을 중심으로 좌우가 대칭된 구조로 이루어져 있고 중앙광장은 고대의 기개를 표현한 호상(虎像)이, 왼편에는 4·19 혁명의 도화선이 된 4·18을 기념하는 4·18 기념비가 자리잡고 있으며 1975년에 개교 70주년을 기념하기 위하여 중앙도서관 신관을 신축하여 1978년 3월에 개관했다. 
    
    # section [17]: [자연계(이공계) 캠퍼스] 
        - summary: 자연은행에서 130억 원을 기부하여 건립된 하나스퀘어를 중심으로 자연계 캠퍼스에는 2006년 9월 완공된 하나스퀘어를 중심으로 공과대학, 이과대학, 정보통신대학, 생명과학대학, 보건과학대학, 정보대학의 교수 연구실 및 강의실이 위치하고 있다. 
    
    # section [18]: [녹지 캠퍼스] 
        - summary: 의과대학과 간호대학의 교수연구실이 위치하고 있고 기숙사와 체육 시설 등이 들어서 있는 녹지 캠퍼스로 불렸던 안암병원은 고려대학교 의료원 산하에 있는 병원 세 곳 중 하나이며, 1971년 12월 고려중앙학원이 우석학원을 합병한 이후 과거 혜화병원을 신축 이전하여 현재에 이르고 있고, 화정체육관 공사가 진행되기 이전에는 노천극장이 폐쇄되면서 녹지운동장으로 옮겨가게 되었다. 
    
    # section [19]: [정릉캠퍼스] 
        - summary: 정릉캠퍼스는 정의관, 진리관, 호림관, 학생회관, KU-MAGIC 연구원 등 5개의 건물로 구성되며 KU-MAGIC Project One을 통해 첨단의료과학센터로 거듭날 예정이다. 
    
    # section [20]: [세종캠퍼스(분교)] 
        - summary: 고특별자치시에 위치한 고려대학교의 분교인 세종캠퍼스는 법적, 행정적으로 엄연히 다른 대학으로 취급되는데 일반적인 의미에서의 '고려대학교'는 서울의 서울캠퍼스만을 가리키는 것으로, 일반적인 의미에서의 '고려대학교'는 서울의 서울캠퍼스만을 가리키는 것으로, 분교인 세종캠퍼스는 법적, 행정적으로 엄연히 다른 대학으로 취급된다. 
    
    # section [21]: [의료원 및 기타 시설] 
        - summary: 고려대학교 의료원 산하에 안암병원, 구로병원, 안산병원 세 곳의 병원이 운영되고 있는 가운데 안암병원에서는 1051병상을 가동하고 있고, 구로병원에서는 1057병상을 가동하고 있는데, 1983년에 개원하여 2008년 증축이 완료됐다. 
    
    # section [22]: [서울캠퍼스 주변 상권] 
        - summary: 학생에는 정문 앞과 제기동 주변이 학교 주변의 가장 주요한 상권이었으나 안암병원의 개원과 더불어 유흥 상권이 형성돼 있는 안암로터리 부근으로 이동하기 시작하고 지하철 6호선 안암역 개통을 계기로 안암로터리 부근의 상권이 안암역 사거리까지 확장되어 현재의 참살이길을 형성하여 정경대학 후문의 24시간 개방이 이루어지기 시작했다. 
    
    # section [23]: [총학생회] 
        - summary: 이처가 총학생회 활동의 지원을 담당하고 있으며 학생 복지를 관장하는 학생복지위원회라는 별도의 기구가 존재하며 총학생회의 기원은 이승만 정권 당시 존재한 중앙학도호국단의 해체와 관련이 깊다. 
    
    # section [24]: [학내 공식 언론단체] 
        - summary: 고47년 창간돼 학생 주도로 발행하는 고려대학교의 주간 교내신문인 고려신문(주간신문) : 1947년 학생 주도로 발행하는 고려대학교의 주간 교내신문으로, 발행비용은 학교 당국이 보조하고 최근 들어 특정 주제에 대한 깊이 있는 기사보다는 학내 뉴스 위주로 1면을 내보내고 있다는 비판이 존재한다. 
    
    # section [25]: [동아리] 
        - summary: 고려대학교의 동아리는 인문계 캠퍼스 학생회관에 동아리방이 있는 중앙동아리와 자연계 캠퍼스 애기능학생회관에 동아리방이 있는 고려대학교 애기능동아리연합회 소속의 애기능중앙동아리로 나뉘며, 두 동아리연합은 동격이다. 
    
    # section [26]: [기타] 
        - summary: 고려대학교 교환학생 교류회(KUBA) : 고려대학교 교환학생 교류회 KUBA는 ‘Korea University Buddy Assistants’의 약자로 외국인 교환학생/방문학생들의 학사와 생활전반에 걸친 정착을 돕는 고려대학교 국제처 산하 자원봉사 단체로, 고려대학교를 전 세계에 알림과 동시에 외국인 학생들이 대한민국과 고려대학교 생활에 빠르게 적응할 수 있도록 도움을 주고, 이를 통해 국경을 초월한 우정과 문화 교류를 쌓는 것을 목적으로 한다. 
    
    # section [27]: [사발식] 
        - summary: 신입발식이란 신입생들이 마시는 막걸리를 선배들이 불러주는 '막걸리찬가'에 맞추어 신입생들이 마시는 행사를 말하며, 고려대학교의 교우로 남기려면 원칙적으로 거쳐야 하는 관문이고 신입생환영회에서 많은 양의 막걸리를 마시도록 하는 것에는 그간의 획일화된 교육과 얽매인 생활의 묵은 때를 모두 토해 비워버리고 학문의 진리와 민족의 정의를 위해 나아가는 고대인이 되라”는 의미가 담겨있다고 한다. 
    
    # section [28]: [동아리 박람회] 
        - summary: 고려대학교 중앙동아리연합회 주최의 동아리 박람회와 애기능동아리연합회 주최의 동아리 박람회가 각각 이틀 간 민주광장과 애기능학생회관 앞에서 열리는데 동아리 구성원들은 박람회 외에도 포스터나 공연, 콘서트, 지인을 통한 여러 홍보활동을 통해 그들이 속한 동아리를 알린다. 
    
    # section [29]: [4·18 기념 마라톤과 구국 대장정] 
        - summary: 매년 4월 18일에는 4·18 의거를 기념하기 위해 4·18 기념 마라톤과 구국대장정을 진행하는데 학교를 출발하여 국립 4·19 묘지를 거쳐 다시 학교로 돌아오는 총 16.2km의 코스로 구성돼 있고 최종 목적지가 국립 4·19 묘지로 설정되어 있는 것에는 4·18이 4·19 정신으로 이어진다는 의미가 담겨 있다. 
    
    # section [30]: [석탑대동제] 
        - summary: ‘년년 5월 중에는 석탑대동제(石塔大同祭)라는 이름의 학생 축제가 있었는데, 이는 대한민국의 민주화를 위한 대학생들이 독재정권 항쟁 과정에서 죽임을 당한 학생들을 추모하면서 살아있는 학샐들의 참여 의식을 고취하기 위한 것이었으며, 1956년 개교기념일엔 대한민국 정부 수립 이후 최초의 민주화 투쟁이 발생하는등 개교 기념일에 여러 가지 이벤트가 발생했다. 
    
    # section [31]: [입실렌티 지·야의 함성] 
        - summary: 입입실렌티 지·야의 함성이란 주로 석탑대동제 마지막 날 저녁을 기해 고려대학교 응원단이 주최하는 응원제로 2011년에 34회째를 맞았으며 매년 고연전 오티를 기해 응원곡 신곡을 발표하고 최근 밝고 경쾌한 분위기의 동작들이 가미된 응원곡들 또한 등장하고 있다. 
    
    # section [32]: [고연전] 
        - summary: 고연전은 연희전문학교와 연희학교의 전신인 연희전문학교가 1925년 5월 30일에 열린 조선체육회 주최 ‘제5회 전조선(全朝鮮) 정구대회’에서 처음으로 맞붙어 애교심을 고양하고 고려대학교와 연세대학교 양교의 친선을 도모하기 위한 목적으로 스포츠 경기를 비롯한 강연, 온라인 게임, 사회공헌활동과 같은 다양한 분야에서 연세대학교와 승부를 겨루는 행사 일체를 의미하지만 정식 명칭은 실질적으로 방송 중계나 신문 보도 등의 공식 석상에만 사용될 뿐이며, 고려대학교에서는 고연전, 연세대학교에서는 연고전이라는 명칭으로 통용된다. 
    
    # section [33]: [운동부] 
        - summary: 운동 종목부 운영으로 여러 종목의 운동부 운영이 가능하며 주 종목 외에도 다양한 종목의 운동부를 운영하고 있다 
    
    # section [34]: [교우회 및 동문] 
        - summary: 고 최초의 대학 동문회인 ‘보전 친목회’로 창립된 고려대학교 교우회는 특별한 유대와 결속을 자랑하는 3대 모임의 하나로 일컬어질 정도로 대한민국 사회에서 상당히 강한 결속력을 발휘하고 있으나 1990년대 후반 입학생들의 구성이 변화함에 따라 현재의 결속력이 향후 약화될 것이라는 관측도 나오고 있다. 
    
    
    


```python
kuki = Kukipedia(target_title = "고려대학교")
wiki_summary_combine_dict = kuki.wiki_summary_combine('ainize', 
                                                      return_dict=True) # 결과를 딕셔너리 형태로 반환한다. 디폴트값은 False.
print(wiki_summary_combine_dict)
```

    {'고려대학교의 모태': '고려대학교의 모태는 대한제국 내장원경 이용익이 1905년에 대한제국 내장원경 이용익이 1905년에 대한제국 내장원경 이용익이 1905년에 설립한 보성전문학교로, 1932년 동아일보 창업자 김성수가 학교를 인수했고 1934년에는 안암동의 현재의 자리로 학교를 옮겼다가 1944년 일제에 의해 학교 이름이 경성척식경제전문학교로 변경됐다.', '종합대학으로의 승격과 우석대학교 의과대학의 흡수·합병': '1945년 해방 후 1945년 해방 후 교명을 보성전문학교로 환원하고, 1946년에 고려대학교로 개칭됐으며 1946년에 종합대학으로의 설립이 인가되고, 1946년에 본교는 애기능 인근 부지를 매입하였으며 1946년에 이공계캠퍼스가 자리 잡는 기반이 된다.', '반복적인 시위와 잦은 휴교 사태': '독60년 4월 11일 3·15 부정선거와 관련된 마산 시위에서 실종된 김주열의 시신이 마산 앞바다에서 발견된 것과 관련하여 수많은 학생들이 ‘마산사건의 책임자를 즉시 처단’할 것을 요구하면서 장외시위를 벌였으나, 돌아오는 길에 신도환의 대한반공청년단 소속 폭력배들에게 피습을 당하여 4·19 혁명의 도화선이 되고 독재정권에 맞선 선배들의 정의로운 행동을 계승하기 위해 ‘4·18 구국대장정’ 행사가 매년 4월 18일에 열린다.', '교육시설의 확충': '1945년 신관을 신축하여 1978년 3월에 개관한 중앙도서관 구관이 1935년에 착공하여 1937년에 개관한 데 이어 1975년에는 창립 70주년을 기념하기 위해 중앙도서관 신관을 신축하여 1978년 3월에 개관하였으며 한편 수도권 인구분산 정책에 따라 조치원분교 설립이 인가됐는데 이것이 현재의 세종캠퍼스가 탄생하는 밑거름이 됐다.', '졸업요건의 변화': '2000년대에 들어서자 세계화 되는 추세에서 고려대학교 졸업자의 실력은 세계적인 수준에 맞추어야 한다는 ‘Global KU 프로젝트’ 등의 영향으로 졸업요건이 대폭 강화되는 등의 변화가 있었고, 2004년 3월 신입생부터는 한자 2급 수준의 졸업요건이 신설됐으나 2011년 들어 일부 학과들이 이 요건을 폐지했으며 2004년 3월 신입생부터는 심화전공이나 이중전공, 연계전공, 이렇게 세 가지 항목 가운데 한 가지를 의무적으로 선택하여 학생설계전공으로 세분화할 수 있는 제2 전공을 의무적으로 이수해야 졸업할 수 있게 됐다.', '개교 100주년 기념사업': '2005년에는 고려대학교가 개교한 지 100주년이 되는 해로 같은 해 5월 5일에는 개교 100주년 기념식이 거행됐고, 2003년에 ‘개교 100주년 기념사업회’가 출범 및 지층에 7천평 규모의 1백주년 기념관에는 박물관과 디지털 도서관을 건립 하는 등 교내에서 다양한 기념사업들이 추진됐다.', '세계 대학 순위 평가': '고윤대 총장의 재임기간 동안 고려대학교는 2006년 영국 《타임즈》지가 발표한 세계대학순위에서 종합 150위에 오르는 등 많은 성과를 기록하였으며, 2009년에는 경영대학 평가 전문기관인 에듀니버설이 전 세계 경영대 학장 1000명을 상대로 한 설문조사에서 ‘추천하고 싶은 대학’ 국내 1위에 올랐으며, 2010년 대한민국 경영대 평가 설문 조사에서 1위를 차지한 데 이어 2012년 UTD 랭킹에서는 86위에 올라 대한민국 대학 중에서 1위를 차지하였고, BK21의 후속 사업인 BK21 플러스의 2013년 초기 선정 결과도 사업단 수와 지원 금액에서 서울대학교에 이어 2위를 차지하였으며, 특히 공과대학의 모든 분야가 BK21 플러스에 선정됐다.', '세계 대학 평가 기관': '고려대학교는 세계에서 가장 유명하고 영향력 있는 3대 세계 대학 평가 기관인 QS가 발표한 세계대학평가 학과별 평가에서 전체 평가 항목 36개 분야 중 20개 학과에서 세계 100위권 이내에 드는 우수한 성과를 보였고, 미국에서 가장 유명한 대학 평가 기관인 US 뉴스&월드 리포트에서는 157위로 대한민국 대학들 중에서는 서울대학교의 뒤를 이어서 2위를 차지했다.', '최근의 세계 대학 평가 결과': '2016년 9월 발표한 QS 세계대학평가에서는 전년 대비 6순위 상승한 98위를 기록하며 대한민국 종합사립대학으로서는 가장 먼저 세계 100위 안에 진입한 대학이 됐고,세계 대학순위 100위권에는 고려대학교 외 세 곳이 포함되었으며 국립 대학인 서울대학교, 한국과학기술원, 포항공과대학교 등이 포함되었으며 2018년에는 전년 대비 4순위 향상한 86위에 등극했다.', '개설 학과·전공': '전부와 대학원 과정이 모두 개설되어 있고 과정이 모두 개설되어 있는 가운데 학부와 대학원 과정이 모두 개설되어 있는 중이다.', '학부 과정': '2022년 기준으로 본교인 서울캠퍼스에는 12개 단과대학, 분교인 세종캠퍼스에는 5개 단과대학이 설치돼 있으며 2009학년도부터 법학전문대학원(로스쿨)이 개원함에 따라 법과대학은 폐지되었으며, 대신 자유전공학부를 신설하여 기존의 법학과 신입생 정원을 대체하고 있고 컴퓨터교육과의 경우 2014년학년도를 시작으로 컴퓨터학과로 개편되면서 더이상 사범대학이 아닌 정보통신대학의 소속으로 변경됐다.', '대학원 과정': '의학대학원은 일반대학원과 전문대학원, 특수대학원로 구분되어 있으며, 의학전문대학원의 경우 전문대학원 53명을 의학과 53명과 병행하여 임시로 운영하였으나 2015년에 폐지 되었고, 2015년에 폐지 되었고, 의학전문대학원의 경우 전문대학원 53명을 의학과 53명과 병행하여 임시로 운영하였으나 2015년에 폐지되었다.', '국제 교류': '국제처에서는 국제 교류의 갈래로 SEP(Student Exchange Program)와 VSP(Visiting Student Program)의 두 종류를 운영하고 있는데, SEP는 본교와 학술 교류 협정이 체결된 외국 대학과 학부 또는 대학원생을 교환하는 프로그램이고, VSP는 소수의 학교와 별도의 협정을 체결하여 본교의 학생들을 대규모로 파견하는 것이 VSP 프로그램의 요지이다.', 'Global KU 프로젝트와 한자졸업요건': '세계 100대 대학에 진입을 목표로 하는 Global KU 프로젝트는 장기적으로 영어강의 확충, 해외 대학과의 교류 증진, 외국인 교수 채용 확대 등을 목표로 하고 있으며, 졸업요건으로 요구하는 토익 점수를 대폭 상승시키고, 2005년 5월에는 개교 100주년을 기념하여 세계대학총장포럼을 개최 하는 등 단기적인 계획도 함께 진행하고 있으며 교내에 도입된 영어강의와 관련해서 학교 측에서는 각 단과대의 특성을 고려해 앞으로 영어로 전공 수업등이 진행되는 영어강의와 관련된 졸업 요건을 단과대별로 완화하거나 강화할 계획인 것으로 알려져 있다.', '제2 전공의 의무화': '제전공 이수학점이 상대적으로 연세대학교보다 10 학점이상 낮기 때문에 연세대학교에서 시행되는 전과 제도를 이용하지 않고도 복수로 전공이 가능하기 때문에 학교 관계자학적·수업지원팀 유신열 과장이 제1전공을 이수하면서 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공과 별도로 이수하였을 경우, 복수전공에 해당하는 과목을 부전공에 해당하는 과목을 부전공의 형태로 이수하였을 경우, 복수전공에 해당하는 학점은 21학점까지 인정받을 수 있다고 말했다.', '캠퍼스 및 학교 소유 시설': '본캠퍼스는 인문·사회계 캠퍼스, 자연계지역, 녹지 캠퍼스, 정릉캠퍼스를 포함한 가운데 어윤대 총장이 재임하던 시기에 3,500억 원의 발전 기금을 유치하고 기업체의 후원을 받아내 교내 전체 건물의 40% 정도를 신축 및 리모델링함에 따라 전체적으로 건물 양식이 일관된 형태를 보이고 있고 어윤대 총장이 재임하던 시기에 3,500억 원의 발전 기금을 유치하고 기업체의 후원을 받아내 교내 전체 건물의 40% 정도를 신축 및 리모델링함에 따라 전체적으로 건물 양식이 일관된 형태를 보이고 있으며 중앙광장과 하나스퀘어가 각각 인문사회계지역과 자연계지역에 위치하여 각 지역의 중심을 이루고 있다.', '인문·사회계 캠퍼스': '인광장과 본관이 위치한 지역으로, 본관 ~ 중앙광장 ~ 정문을 중심으로 좌우가 대칭된 구조로 이루어져 있고 중앙광장은 고대의 기개를 표현한 호상(虎像)이, 왼편에는 4·19 혁명의 도화선이 된 4·18을 기념하는 4·18 기념비가 자리잡고 있으며 1975년에 개교 70주년을 기념하기 위하여 중앙도서관 신관을 신축하여 1978년 3월에 개관했다.', '자연계(이공계) 캠퍼스': '자연은행에서 130억 원을 기부하여 건립된 하나스퀘어를 중심으로 자연계 캠퍼스에는 2006년 9월 완공된 하나스퀘어를 중심으로 공과대학, 이과대학, 정보통신대학, 생명과학대학, 보건과학대학, 정보대학의 교수 연구실 및 강의실이 위치하고 있다.', '녹지 캠퍼스': '의과대학과 간호대학의 교수연구실이 위치하고 있고 기숙사와 체육 시설 등이 들어서 있는 녹지 캠퍼스로 불렸던 안암병원은 고려대학교 의료원 산하에 있는 병원 세 곳 중 하나이며, 1971년 12월 고려중앙학원이 우석학원을 합병한 이후 과거 혜화병원을 신축 이전하여 현재에 이르고 있고, 화정체육관 공사가 진행되기 이전에는 노천극장이 폐쇄되면서 녹지운동장으로 옮겨가게 되었다.', '정릉캠퍼스': '정릉캠퍼스는 정의관, 진리관, 호림관, 학생회관, KU-MAGIC 연구원 등 5개의 건물로 구성되며 KU-MAGIC Project One을 통해 첨단의료과학센터로 거듭날 예정이다.', '세종캠퍼스(분교)': "고특별자치시에 위치한 고려대학교의 분교인 세종캠퍼스는 법적, 행정적으로 엄연히 다른 대학으로 취급되는데 일반적인 의미에서의 '고려대학교'는 서울의 서울캠퍼스만을 가리키는 것으로, 일반적인 의미에서의 '고려대학교'는 서울의 서울캠퍼스만을 가리키는 것으로, 분교인 세종캠퍼스는 법적, 행정적으로 엄연히 다른 대학으로 취급된다.", '의료원 및 기타 시설': '고려대학교 의료원 산하에 안암병원, 구로병원, 안산병원 세 곳의 병원이 운영되고 있는 가운데 안암병원에서는 1051병상을 가동하고 있고, 구로병원에서는 1057병상을 가동하고 있는데, 1983년에 개원하여 2008년 증축이 완료됐다.', '서울캠퍼스 주변 상권': '학생에는 정문 앞과 제기동 주변이 학교 주변의 가장 주요한 상권이었으나 안암병원의 개원과 더불어 유흥 상권이 형성돼 있는 안암로터리 부근으로 이동하기 시작하고 지하철 6호선 안암역 개통을 계기로 안암로터리 부근의 상권이 안암역 사거리까지 확장되어 현재의 참살이길을 형성하여 정경대학 후문의 24시간 개방이 이루어지기 시작했다.', '총학생회': '이처가 총학생회 활동의 지원을 담당하고 있으며 학생 복지를 관장하는 학생복지위원회라는 별도의 기구가 존재하며 총학생회의 기원은 이승만 정권 당시 존재한 중앙학도호국단의 해체와 관련이 깊다.', '학내 공식 언론단체': '고47년 창간돼 학생 주도로 발행하는 고려대학교의 주간 교내신문인 고려신문(주간신문) : 1947년 학생 주도로 발행하는 고려대학교의 주간 교내신문으로, 발행비용은 학교 당국이 보조하고 최근 들어 특정 주제에 대한 깊이 있는 기사보다는 학내 뉴스 위주로 1면을 내보내고 있다는 비판이 존재한다.', '동아리': '고려대학교의 동아리는 인문계 캠퍼스 학생회관에 동아리방이 있는 중앙동아리와 자연계 캠퍼스 애기능학생회관에 동아리방이 있는 고려대학교 애기능동아리연합회 소속의 애기능중앙동아리로 나뉘며, 두 동아리연합은 동격이다.', '기타': '고려대학교 교환학생 교류회(KUBA) : 고려대학교 교환학생 교류회 KUBA는 ‘Korea University Buddy Assistants’의 약자로 외국인 교환학생/방문학생들의 학사와 생활전반에 걸친 정착을 돕는 고려대학교 국제처 산하 자원봉사 단체로, 고려대학교를 전 세계에 알림과 동시에 외국인 학생들이 대한민국과 고려대학교 생활에 빠르게 적응할 수 있도록 도움을 주고, 이를 통해 국경을 초월한 우정과 문화 교류를 쌓는 것을 목적으로 한다.', '사발식': "신입발식이란 신입생들이 마시는 막걸리를 선배들이 불러주는 '막걸리찬가'에 맞추어 신입생들이 마시는 행사를 말하며, 고려대학교의 교우로 남기려면 원칙적으로 거쳐야 하는 관문이고 신입생환영회에서 많은 양의 막걸리를 마시도록 하는 것에는 그간의 획일화된 교육과 얽매인 생활의 묵은 때를 모두 토해 비워버리고 학문의 진리와 민족의 정의를 위해 나아가는 고대인이 되라”는 의미가 담겨있다고 한다.", '동아리 박람회': '고려대학교 중앙동아리연합회 주최의 동아리 박람회와 애기능동아리연합회 주최의 동아리 박람회가 각각 이틀 간 민주광장과 애기능학생회관 앞에서 열리는데 동아리 구성원들은 박람회 외에도 포스터나 공연, 콘서트, 지인을 통한 여러 홍보활동을 통해 그들이 속한 동아리를 알린다.', '4·18 기념 마라톤과 구국 대장정': '매년 4월 18일에는 4·18 의거를 기념하기 위해 4·18 기념 마라톤과 구국대장정을 진행하는데 학교를 출발하여 국립 4·19 묘지를 거쳐 다시 학교로 돌아오는 총 16.2km의 코스로 구성돼 있고 최종 목적지가 국립 4·19 묘지로 설정되어 있는 것에는 4·18이 4·19 정신으로 이어진다는 의미가 담겨 있다.', '석탑대동제': '‘년년 5월 중에는 석탑대동제(石塔大同祭)라는 이름의 학생 축제가 있었는데, 이는 대한민국의 민주화를 위한 대학생들이 독재정권 항쟁 과정에서 죽임을 당한 학생들을 추모하면서 살아있는 학샐들의 참여 의식을 고취하기 위한 것이었으며, 1956년 개교기념일엔 대한민국 정부 수립 이후 최초의 민주화 투쟁이 발생하는등 개교 기념일에 여러 가지 이벤트가 발생했다.', '입실렌티 지·야의 함성': '입입실렌티 지·야의 함성이란 주로 석탑대동제 마지막 날 저녁을 기해 고려대학교 응원단이 주최하는 응원제로 2011년에 34회째를 맞았으며 매년 고연전 오티를 기해 응원곡 신곡을 발표하고 최근 밝고 경쾌한 분위기의 동작들이 가미된 응원곡들 또한 등장하고 있다.', '고연전': '고연전은 연희전문학교와 연희학교의 전신인 연희전문학교가 1925년 5월 30일에 열린 조선체육회 주최 ‘제5회 전조선(全朝鮮) 정구대회’에서 처음으로 맞붙어 애교심을 고양하고 고려대학교와 연세대학교 양교의 친선을 도모하기 위한 목적으로 스포츠 경기를 비롯한 강연, 온라인 게임, 사회공헌활동과 같은 다양한 분야에서 연세대학교와 승부를 겨루는 행사 일체를 의미하지만 정식 명칭은 실질적으로 방송 중계나 신문 보도 등의 공식 석상에만 사용될 뿐이며, 고려대학교에서는 고연전, 연세대학교에서는 연고전이라는 명칭으로 통용된다.', '운동부': '운동 종목부 운영으로 여러 종목의 운동부 운영이 가능하며 주 종목 외에도 다양한 종목의 운동부를 운영하고 있다', '교우회 및 동문': '고 최초의 대학 동문회인 ‘보전 친목회’로 창립된 고려대학교 교우회는 특별한 유대와 결속을 자랑하는 3대 모임의 하나로 일컬어질 정도로 대한민국 사회에서 상당히 강한 결속력을 발휘하고 있으나 1990년대 후반 입학생들의 구성이 변화함에 따라 현재의 결속력이 향후 약화될 것이라는 관측도 나오고 있다.'}
    

<br>

### 6. make_wordcloud(summary_model, backgroundcolor, width, height)

`wiki_summary_combine()` 함수로 요약된 결과물을 워드클라우드로 표현해주는 함수이다.

- `backgroundcolor`: 워드클라우드의 배경색 지정(default='white')
- `width`: 워드클라우드의 가로 길이 지정(default=600)
- `height`: 워드클라우드의 세로 길이 지정(default=400)


```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.make_wordcloud('ainize', backgroundcolor='white', width=600, height=400)
```


    
<img src="https://user-images.githubusercontent.com/115082062/229094458-35aa38f6-74c7-46ca-9e75-f49475d63a90.png">

<br>


    



```python
kuki = Kukipedia(target_title = "손흥민")
kuki.make_wordcloud('ainize', backgroundcolor='white', width=600, height=400)
```


<img arc="https://user-images.githubusercontent.com/115082062/229094555-fd44b6df-1999-4ba9-acf1-e112f63df0ed.png">

<br>


그런데 요약문을 워드클라우드로 나타내는 것보다 요약 안 된 전체 텍스트를 워드클라우드하는 게 더 나은 것 같긴 하다.

`make_wordcloud_entire`은 요약 안하고 원본 전체 텍스트를 워드클라우드로 나타내는 함수이다.


```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.make_wordcloud_entire()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229094684-c119e520-9e06-4239-95e2-679f47f9ff5a.png">
    

<br>

### 7. topic_modeling(summary_model)
요약문을 토대로 토픽 모델링을 해주는 함수이다. 최적의 토픽 개수를 함수 내에서 찾아서 학습한다.

워드클라우드와 마찬가지의 이유로 전체 text를 토대로 토픽 모델링을 하는 `topic_modeling_entire`도 있다.


```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.topic_modeling('ainize')
```


<img src="https://user-images.githubusercontent.com/115082062/229095546-ac8670e0-d844-4085-aac2-dd5d0eff982b.png">



<br>



```python
kuki = Kukipedia(target_title = "고려대학교")
kuki.topic_modeling_entire()
```
<img src="https://user-images.githubusercontent.com/115082062/229095678-f5eac3d8-ea39-4799-8247-3417e55bb4af.png">
 