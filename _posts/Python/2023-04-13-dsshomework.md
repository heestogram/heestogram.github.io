---
layout: single
title: "깃허브 블로그와 깃허브 커밋 기록 크롤링하기"
excerpt: "개인 깃허브 블로그에 올린 글 제목, 카테고리를 크롤링하고, 깃허브 레퍼지토리의 커밋 기록을 크롤링해보자"

published: true

categories:
  - Python

tags: [python, Crawling, selenium, BeautifulSoup]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2023-04-13 01:20:00
last_modified_at: 2023-04-13 01:20:00
---

<br>

해당 포스팅은 고려대학교 23년도 1학기 신은경 교수님의 '데이터사이언스와 사회학' 수업 과제의 일환이다.

<br>

## 1. import

우선 크롤링에 필요한 라이브러리를 import한다.

```python
import requests
from bs4 import BeautifulSoup as bs
import pandas as pd
from datetime import datetime, timedelta
```


<br>

## 2. 깃허브 브로그 크롤링

크롤링할 블로그의 주소를 적는다. 이때, 모든 글이 한 페이지에 모여져 있는 page-title 창을 활용한다. 이는 크롤링하기에 용이하게 하기 위함이다.

또한 "archive"라는 class를 찾아주기 위해 `beautifulsoup`의 함수인 `find`를 사용한다. "archive" class에는 내가 포스팅한 모든 글들이 모여있다.


```python
page = requests.get("https://heestogram.github.io/categories/#page-title")
soup = bs(page.text, "html.parser")
soup = soup.find(class_="archive")
```

<img src="https://user-images.githubusercontent.com/115082062/231677085-25c788eb-a0ec-4548-b131-35b9d3b555c1.png">

위 사진을 통해 알 수 있듯이, 각 포스팅에 대한 text 정보는 `archive__item` class에 들어가있다. 이 class를 더욱 세분화해서 들어가면, `archive__item-title no_toc` class는 제목을, `archive__item-excerpt`은 게시 날짜와 category를, `description`은 글의 요약 정보를 알려주고 있다.

<br>


```python
dict_ = {}
title_list = []
date_list = []
category_list = []
summary_list = []
num_of_posts = len(soup.find_all(class_='archive__item-title no_toc'))

# 총 포스팅 개수만큼 반복문
for i in range(num_of_posts):
    step1 = soup.find_all(class_='archive__item-title no_toc')[i]
    step2 = step1.find('a').text[:-1]
    title_list.append(step2)
    
    step1 = soup.find_all(class_='archive__item-excerpt')[2*i].text
    
    date_list.append(datetime.strptime(step1.split()[0], '%m/%d/%Y').strftime("%Y-%m-%d"))
    category_list.append(step1.split()[1])
    
    step1 = soup.find_all(itemprop='description')[i].text[:-1]
    summary_list.append(step1)
    
dict_['title'] = title_list
dict_['date'] = date_list
dict_['category'] = category_list
dict_['summary'] = summary_list
```


```python
df_ = pd.DataFrame(dict_)
```


```python
df_commit.to_csv('github_blog_crawling.csv')
```

<br>

## 3. 깃허브 커밋 기록 크롤링

깃허브 내에서 파일을 올리거나, 수정하는 모든 행위는 commit이라는 하나의 단위가 된다. 따라서 commit이 많을수록 활발하게 깃허브를 가꾸었다고 짐작할 수 있다.

커밋 기록을 크롤링하는 건 더 간단하다.

<img src="https://user-images.githubusercontent.com/115082062/231678479-8d0528a2-a41a-447b-a49b-6f1425c5d34a.png">

위 사진처럼, 격자형태로 날짜별로 커밋 정도가 초록색으로 표시되어있다. 하나의 작은 사각형이 하루를 뜻한다. 이 사각형에 칠해진 초록색이 짙어질수록, 해당 날짜에 많은 커밋을 했다는 의미이다.

각 사각형은 해당 날짜에 몇 개의 커밋을 했다는 text를 가지고 있다.

따라서 이 모든 사각형을 `find_all`함수로 잡아내고 text만 추출하면 어느날 몇 개의 커밋을 했는지 알 수 있다.

<br>


```python
page = requests.get("https://github.com/heestogram")
soup = bs(page.text, "html.parser")
soup = soup.find_all(class_="ContributionCalendar-day")
```


```python
text_list = []
date_list = []
dict_ = {}
length = len(soup)
for i in range(length):
    text_list.append(soup[i].text)
    try:
        date_list.append(soup[i]['data-date'])
    except KeyError:
        date_list.append('no date')
```


```python
dict_['date']=date_list
dict_['contribution']=text_list
df_commit = pd.DataFrame(dict_)
```




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
      <th>date</th>
      <th>contribution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-27</td>
      <td>No contributions on Sunday, March 27, 2022</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-03-28</td>
      <td>No contributions on Monday, March 28, 2022</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-03-29</td>
      <td>No contributions on Tuesday, March 29, 2022</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-30</td>
      <td>No contributions on Wednesday, March 30, 2022</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-03-31</td>
      <td>No contributions on Thursday, March 31, 2022</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>371</th>
      <td>no date</td>
      <td></td>
    </tr>
    <tr>
      <th>372</th>
      <td>no date</td>
      <td></td>
    </tr>
    <tr>
      <th>373</th>
      <td>no date</td>
      <td></td>
    </tr>
    <tr>
      <th>374</th>
      <td>no date</td>
      <td></td>
    </tr>
    <tr>
      <th>375</th>
      <td>no date</td>
      <td></td>
    </tr>
  </tbody>
</table>
<p>376 rows × 2 columns</p>
</div>




```python
df_commit = df_commit[df_commit.date != 'no date']
```




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
      <th>date</th>
      <th>contribution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-27</td>
      <td>No contributions on Sunday, March 27, 2022</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-03-28</td>
      <td>No contributions on Monday, March 28, 2022</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-03-29</td>
      <td>No contributions on Tuesday, March 29, 2022</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-30</td>
      <td>No contributions on Wednesday, March 30, 2022</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-03-31</td>
      <td>No contributions on Thursday, March 31, 2022</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>366</th>
      <td>2023-03-28</td>
      <td>No contributions on Tuesday, March 28, 2023</td>
    </tr>
    <tr>
      <th>367</th>
      <td>2023-03-29</td>
      <td>No contributions on Wednesday, March 29, 2023</td>
    </tr>
    <tr>
      <th>368</th>
      <td>2023-03-30</td>
      <td>1 contribution on Thursday, March 30, 2023</td>
    </tr>
    <tr>
      <th>369</th>
      <td>2023-03-31</td>
      <td>11 contributions on Friday, March 31, 2023</td>
    </tr>
    <tr>
      <th>370</th>
      <td>2023-04-01</td>
      <td>No contributions on Saturday, April 1, 2023</td>
    </tr>
  </tbody>
</table>
<p>371 rows × 2 columns</p>
</div>




```python
df_commit['date'] = pd.to_datetime(df_commit['date'])
```

    C:\Users\희준\AppData\Local\Temp\ipykernel_25104\184324504.py:1: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      df_commit['date'] = pd.to_datetime(df_commit['date'])
    


```python
df_commit.to_csv('github_commit_crawling.csv')
```

<br>