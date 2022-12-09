---
layout: single
title: "[웹 크롤링] statiz 사이트에서 2022시즌 타자 기록 크롤링하기 (feat. selenium, 크롬 웹 드라이버, BeautifulSoup)"
excerpt: "selenium, BeautifulSoup을 이용하여 statiz 사이트의 2022시즌 타자 기록들을 크롤링해온다."

published: true

categories:
  - Python

tags: [python, Crawling, selenium, BeautifulSoup, baseball, statiz]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2022-12-09 14:50:00
last_modified_at: 2022-12-09 14:50:00
---

<br>

baseball 카테고리에서 similarity scores에 관한 글을 올렸었다. 해당 글에서 만든 엑셀 파일의 데이터를 스탯티즈에서 크롤링해왔는데, 그 방법에 대해 소개하는 글을 써보려한다.

아래 글 참고

[[야구 탈럼] Similarity Scores로 닮은꼴 선수 찾아내기 + 스탯티즈 크롤링해서 직접 계산기 만들기](https://heestogram.github.io/baseball/baseball-similarity/)

<br>

## 크롬드라이버, selenium, BeautifulSoup 설치

우선 웹 크롤링에 필요한 사전준비를 하자.
<br>
가장 먼저 크롬 웹 드라이버를 설치해야 한다. 아래 링크로 들어가서 자신의 크롬 버전에 알맞은 드라이버를 설치해주면 된다. 버전 호환이 안 될 시 실행이 되지 않는다.

[ChromeDriver 설치 링크](https://chromedriver.chromium.org/downloads)

본인의 크롬 버전을 알고 싶다면 주소 창에 chrome://version을 붙여넣어 보면 확인할 수 있다.

이제 본격적인 모듈 설치를 해보자.

- `selenium`: 웹 브라우저 상에서 동적인 움직임을 제어할 수 있게 도와주는 프레임워크. 크롬드라이버를 사용할 수 있게 한다.
- `BeautifulSoup`: html과 xml 문서를 parsing하는 패키지

이 두가지 모듈을 조합하여 웹 크롤링을 수행해볼 것이다.


```python
!pip install selenium
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
```

이제 크롬 드라이버를 실행해볼 것이다. `excutable_path`에는 앞서 설치한 크롬드라이버의 디렉터리 경로를 적어주면 된다.

```python
driver = webdriver.Chrome(executable_path = "C:/Users/희준/downloads/chromedriver_win32/chromedriver.exe")
```

<br>
    
## 스탯티즈 크롤링 전체 코드

아래 코드가 전체 코드이다. 

```python
for i in range(4):
    url = 'http://www.statiz.co.kr/stat.php?mid=stat&re=0&ys=2022&ye=2022&sn=100&pa={}'.format(i*100)
    driver.get(url)
    driver.implicitly_wait(time_to_wait=5)
    html = driver.find_element(By.XPATH,'//*[@id="mytable"]/tbody').get_attribute("innerHTML")
    soup = BeautifulSoup(html, 'html.parser')

    temp = [i.text.strip() for i in soup.findAll("tr")] #tr태그에서 text만 저장하고 공백 제거
    temp = pd.Series(temp) #list 객체를 series 객체로 변경
    
    #중간중간에 '순'이나 'WAR'로 시작하는 행들이 있는데 이를 제거해준다
    #그리고 index를 reset
    temp = temp[~temp.str.match("[순W]")].reset_index(drop=True)
    
    #띄어쓰기 기준으로 분류해서 데이터프레임으로 만들기
    temp = temp.apply(lambda x: pd.Series(x.split(' ')))
    
    #선수 팀 정보 이후 첫번째 기록과는 space 하나로 구분, 그 이후로는 space 두개로 구분이 되어 있음 
    #그래서 space 하나로 구분을 시키면, 빈 column들이 존재 하는데, 해당 column들 제거 
    temp = temp.replace('', np.nan).dropna(axis=1) 
    
    #WAR이 두 열이나 존재해서 처음 나오는 WAR열을 삭제. 1열에 있음
    temp = temp.drop(1,axis=1)
    
    #선수 이름 앞의 숫자 제거
    temp[0] = temp[0].str.replace("^\d+", "")
    
    #선수들의 생일 정보가 담긴 tag들 가져오기
    birth = [i.find("a") for i in soup.findAll('tr') if 'birth' in i.find('a').attrs['href']]
    
    #tag에서 생일만 추출하기
    p = re.compile("\d{4}")
    birth = [p.findall(i.attrs['href'])[0] for i in birth]
    
    temp['생일']=birth
    
    if i ==0:
        result = temp
    else:
        result = result.append(temp)
        result = result.reset_index(drop=True)
    print(i, "완료")
    
columns = ['선수'] + [i.text for i in soup.findAll("tr")[0].findAll("th")][4:-3] + ['타율','출루율','장타율','OPS','wOBA','wRC+','WAR+','WPA','생년']
result.columns = columns

driver.close()

#선수 이름을 슬라이싱하여 새로운 열로 추가
result['이름'] = result['선수'].map(lambda x:x[:x.find('22')])
#선수 포지션을 슬라이싱하여 새로운 열로 추가
result['포지션'] = result['선수'].map(lambda x:x[x.find('22')+3:])

# 투수교체나 대타 기용 과정에서 타석에 들어선 것으로 처리되는 투수들이 간혹 있음
# 이 투수들의 row를 삭제해주기 위한 과정
pitcher_index = result[result['포지션']=='P'].index
result.drop(pitcher_index, inplace=True)

```

주석을 달아놓아서 이해에 큰 어려움은 없겠지만, 좀 더 세분화하여 설명해보도록 하자.

<br>

## 스탯티즈 사이트에 접근

url로 지정한 부분에서 ys, ye 부분을 주목해보자. ys는 검색하고자 하는 시즌 시작년도이고, ye는 시즌 종료년도이다. 필자는 2022시즌 데이터가 필요하므로 시작과 끝을 2022로 지정했다. sn은 한 페이지에 몇 명의 선수씩 불러올 것인지를 지정한다. pa는 몇 번째 선수부터 시작할지이다. 즉, sn=100, pa=i로 하고 반복문을 4번 돌렸으므로 1~100번째 선수가 크롤링되고, 101~200번째 선수가 크롤링 되는 식으로 이어지다 400번째 선수까지 크롤링될 것이다. 2022시즌 한 타석이라도 들어선 선수는 3xx명이므로 반복문을 4회로 설정했다.
```python
for i in range(4):
    url = 'http://www.statiz.co.kr/stat.php?mid=stat&re=0&ys=2022&ye=2022&sn=100&pa={}'.format(i*100)
```

<br>

## get 함수

`get()` 함수는 검색하고자 하는 url을 불러오는 역할을 한다.
```python
    driver.get(url)
    driver.implicitly_wait(time_to_wait=5)
    html = driver.find_element(By.XPATH,'//*[@id="mytable"]/tbody').get_attribute("innerHTML")
    soup = BeautifulSoup(html, 'html.parser')
```

**이 어 서 작 성 할 것**


```python
result
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
      <th>선수</th>
      <th>G</th>
      <th>타석</th>
      <th>타수</th>
      <th>득점</th>
      <th>안타</th>
      <th>2타</th>
      <th>3타</th>
      <th>홈런</th>
      <th>루타</th>
      <th>...</th>
      <th>출루율</th>
      <th>장타율</th>
      <th>OPS</th>
      <th>wOBA</th>
      <th>wRC+</th>
      <th>WAR+</th>
      <th>WPA</th>
      <th>생년</th>
      <th>이름</th>
      <th>포지션</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>이정후22키CF</td>
      <td>142</td>
      <td>627</td>
      <td>553</td>
      <td>85</td>
      <td>193</td>
      <td>36</td>
      <td>10</td>
      <td>23</td>
      <td>318</td>
      <td>...</td>
      <td>.421</td>
      <td>.575</td>
      <td>.996</td>
      <td>.441</td>
      <td>182.5</td>
      <td>9.23</td>
      <td>6.72</td>
      <td>1998</td>
      <td>이정후</td>
      <td>CF</td>
    </tr>
    <tr>
      <th>1</th>
      <td>피렐라22삼LF</td>
      <td>141</td>
      <td>630</td>
      <td>561</td>
      <td>102</td>
      <td>192</td>
      <td>33</td>
      <td>4</td>
      <td>28</td>
      <td>317</td>
      <td>...</td>
      <td>.411</td>
      <td>.565</td>
      <td>.976</td>
      <td>.434</td>
      <td>169.3</td>
      <td>7.40</td>
      <td>4.20</td>
      <td>1989</td>
      <td>피렐라</td>
      <td>LF</td>
    </tr>
    <tr>
      <th>2</th>
      <td>나성범22KRF</td>
      <td>144</td>
      <td>649</td>
      <td>563</td>
      <td>92</td>
      <td>180</td>
      <td>39</td>
      <td>2</td>
      <td>21</td>
      <td>286</td>
      <td>...</td>
      <td>.402</td>
      <td>.508</td>
      <td>.910</td>
      <td>.411</td>
      <td>157.4</td>
      <td>6.50</td>
      <td>3.74</td>
      <td>1989</td>
      <td>나성범</td>
      <td>RF</td>
    </tr>
    <tr>
      <th>3</th>
      <td>오지환22LSS</td>
      <td>142</td>
      <td>569</td>
      <td>494</td>
      <td>75</td>
      <td>133</td>
      <td>16</td>
      <td>4</td>
      <td>25</td>
      <td>232</td>
      <td>...</td>
      <td>.357</td>
      <td>.470</td>
      <td>.827</td>
      <td>.372</td>
      <td>138.6</td>
      <td>5.77</td>
      <td>2.56</td>
      <td>1990</td>
      <td>오지환</td>
      <td>SS</td>
    </tr>
    <tr>
      <th>4</th>
      <td>최정22S3B</td>
      <td>121</td>
      <td>505</td>
      <td>414</td>
      <td>80</td>
      <td>110</td>
      <td>21</td>
      <td>0</td>
      <td>26</td>
      <td>209</td>
      <td>...</td>
      <td>.386</td>
      <td>.505</td>
      <td>.891</td>
      <td>.400</td>
      <td>145.4</td>
      <td>5.15</td>
      <td>3.72</td>
      <td>1987</td>
      <td>최정</td>
      <td>3B</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>352</th>
      <td>조세진22롯RF</td>
      <td>39</td>
      <td>88</td>
      <td>86</td>
      <td>6</td>
      <td>16</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>19</td>
      <td>...</td>
      <td>.195</td>
      <td>.221</td>
      <td>.416</td>
      <td>.192</td>
      <td>5.7</td>
      <td>-0.76</td>
      <td>-0.64</td>
      <td>2003</td>
      <td>조세진</td>
      <td>RF</td>
    </tr>
    <tr>
      <th>353</th>
      <td>유로결22한LF</td>
      <td>30</td>
      <td>68</td>
      <td>60</td>
      <td>5</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>8</td>
      <td>...</td>
      <td>.200</td>
      <td>.133</td>
      <td>.333</td>
      <td>.164</td>
      <td>-11.4</td>
      <td>-0.82</td>
      <td>-1.05</td>
      <td>2000</td>
      <td>유로결</td>
      <td>LF</td>
    </tr>
    <tr>
      <th>354</th>
      <td>정보근22롯C</td>
      <td>95</td>
      <td>226</td>
      <td>199</td>
      <td>8</td>
      <td>38</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>43</td>
      <td>...</td>
      <td>.250</td>
      <td>.216</td>
      <td>.466</td>
      <td>.218</td>
      <td>24.1</td>
      <td>-0.91</td>
      <td>-1.78</td>
      <td>1999</td>
      <td>정보근</td>
      <td>C</td>
    </tr>
    <tr>
      <th>355</th>
      <td>박경수22K2B</td>
      <td>100</td>
      <td>194</td>
      <td>166</td>
      <td>13</td>
      <td>20</td>
      <td>3</td>
      <td>0</td>
      <td>3</td>
      <td>32</td>
      <td>...</td>
      <td>.234</td>
      <td>.193</td>
      <td>.427</td>
      <td>.213</td>
      <td>21.9</td>
      <td>-1.01</td>
      <td>-2.16</td>
      <td>1984</td>
      <td>박경수</td>
      <td>2B</td>
    </tr>
    <tr>
      <th>356</th>
      <td>김헌곤22삼CF</td>
      <td>80</td>
      <td>239</td>
      <td>224</td>
      <td>18</td>
      <td>43</td>
      <td>8</td>
      <td>0</td>
      <td>1</td>
      <td>54</td>
      <td>...</td>
      <td>.224</td>
      <td>.241</td>
      <td>.465</td>
      <td>.217</td>
      <td>24.6</td>
      <td>-1.58</td>
      <td>-2.24</td>
      <td>1988</td>
      <td>김헌곤</td>
      <td>CF</td>
    </tr>
  </tbody>
</table>
<p>289 rows × 31 columns</p>
</div>



```python
#크롤링한 파일 저장
result.to_excel('statiz_crawl_2022.xlsx', index=False)
```
