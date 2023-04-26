---
layout: single
title: "에브리타임 원하는 게시판 본문과 댓글 등 크롤링"
excerpt: "에브리타임 웹페이지에서 원하는 게시판의 제목, 본문, 댓글, 좋아요, 스크랩, 날짜 등을 크롤링해오자"

published: true

categories:
  - Python

tags: [python, Crawling, selenium, BeautifulSoup]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2023-04-26 01:20:00
last_modified_at: 2023-04-26 01:20:00
---

<br>

해당 포스팅은 고려대학교 23년도 1학기 신은경 교수님의 '데이터사이언스와 사회학' 수업 과제의 일환이다.

<br>

## 1. 라이브러리 import
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import pandas as pd
from datetime import datetime, timedelta
```

<br>

## 2. 에브리타임 로그인

우선 자신의 크롬 버전에 알맞은 chromedriver를 설치해야 한다. 설치를 했다면 해당 경로를 아래에 잘 기입해준다.

```python
driver = webdriver.Chrome(executable_path = "D:/heejun/dev/chromedriver.exe")
```

<br>

이제 로그인을 하여 고려대학교 에브리타임에 접속을 해야한다.

아이디와 비밀번호까지는 send_keys가 잘 되는데, 로봇 검사가 중간에 뜨기 때문에 `click()`까지는 자동 실행이 안 된다. 이 부분은 수동으로 클릭 해주도록 한다.

```python
driver.implicitly_wait(1)
driver.get('https://everytime.kr/login')
driver.find_element(By.NAME,'userid').send_keys('아이디')
driver.find_element(By.NAME,'password').send_keys('비밀번호')
driver.find_element(By.XPATH,'//*[@class="submit"]/input').click()
driver.implicitly_wait(1)
```

## 3. 크롤링 코드

```python
dictionary={} # 크롤링한 text들을 담기 위한 dictionary
page = 0 #페이지 순환을 위한 count num

content_=[] # 글 본문 담을 리스트
comment_=[] # 댓글 담을 리스트
like_=[] # 좋아요 수 담을 리스트
comment_count_=[] # 댓글 수 담을 리스트
scrap_=[] # 스크랩 수 담을 리스트
time_=[]

while True:
    print('page '+str(page))
    
    if page > 100: #100페이지 넘어가면 반복문 중단
        break

    page = page+1
    
    driver.get("https://everytime.kr/hotarticle/p/{}".format(str(page)))
    driver.implicitly_wait(1)

    # 게시물에 해당하는 클래스들을 추출
    posts = driver.find_elements(By.CSS_SELECTOR, 'article > a.article')
    # 해당 클래스들의 링크들을 추출
    links = [post.get_attribute('href') for post in posts]

    # 모아놓은 링크들을 순회(=각 게시물에 들어가서 정보들 추출)
    for link in links:
        driver.get(link)
        comment_list = []
        texts = driver.find_elements(By.CSS_SELECTOR, 'p.large')

        title_text= driver.find_element(By.CSS_SELECTOR, 'h2.large').text

        like_.append(driver.find_element(By.CSS_SELECTOR, 'li.vote').text)
        comment_count_.append(driver.find_element(By.CSS_SELECTOR, 'li.comment').text)
        scrap_.append(driver.find_element(By.CSS_SELECTOR, 'li.scrap').text)
        time_.append(driver.find_element(By.CSS_SELECTOR, 'time.large').text)

        i = 0
        for comment in texts:
            if i==0:
                content_.append(title_text+' '+comment.text)
            else:
                comment_list.append(comment.text)
            i=i+1

        comment_.append(comment_list)
```

<br>

원래 처음 목표했던 바는 취업게시판과 졸업생게시판을 순회하며 크롤링을 해와서 진로, 취업 문제에 대한 데이터셋을 마련하는 것이었다. 이렇듯 여러 게시판을 순회하는 반복문은 아래처럼 작성할 수 있다.

```python
dictionary={} # 크롤링한 text들을 담기 위한 dictionary
page = 0 #페이지 순환을 위한 count num

content_=[] # 글 본문 담을 리스트
comment_=[] # 댓글 담을 리스트
like_=[] # 좋아요 수 담을 리스트
comment_count_=[] # 댓글 수 담을 리스트
scrap_=[] # 스크랩 수 담을 리스트
time_=[]

while True:
    print('page '+str(page))
    
    if page > 199: # 200페이지 넘어가면 반복문 종료
        break

    page = page+1
    
    board = ['370507','385968'] # 취업,진로 게시판과 졸업생 게시판 코드
    for num in board: # 두 게시판을 순회
        driver.get("https://everytime.kr/{}/p/{}".format(num, str(page)))
        driver.implicitly_wait(1)
        
        # 게시물에 해당하는 클래스들을 추출
        posts = driver.find_elements(By.CSS_SELECTOR, 'article > a.article')
        # 해당 클래스들의 링크들을 추출
        links = [post.get_attribute('href') for post in posts]

        # 모아놓은 링크들을 순회(=각 게시물에 들어가서 정보들 추출)
        for link in links:
            driver.get(link)
            comment_list = []
            texts = driver.find_elements(By.CSS_SELECTOR, 'p.large')
            
            title_text= driver.find_element(By.CSS_SELECTOR, 'h2.large').text

            like_.append(driver.find_element(By.CSS_SELECTOR, 'li.vote').text)
            comment_count_.append(driver.find_element(By.CSS_SELECTOR, 'li.comment').text)
            scrap_.append(driver.find_element(By.CSS_SELECTOR, 'li.scrap').text)
            time_.append(driver.find_element(By.CSS_SELECTOR, 'time.large').text)

            i = 0
            for comment in texts:
                if i==0:
                    content_.append(title_text+' '+comment.text)
                else:
                    comment_list.append(comment.text)
                i=i+1

            comment_.append(comment_list)
```

<br>

## 크롤링해온 것 데이터프레임으로 병합

```python
def make_df():
    dictionary['content']=content_
    dictionary['comment']=comment_
    dictionary['like']=like_
    dictionary['comment_count']=comment_count_
    dictionary['scrap']=scrap_
    dictionary['date']=time_
    df = pd.DataFrame(dictionary)
    return df

# 올해게시글은 연도 없이 날짜와 시간만 올라오고,
# 1년 전 게시글은 연도와 날짜, 시간이 같이 올라오므로 이를 통일하기 위한 함수
def date_preprocess(df):
    for i in range(len(df)):
        if len(df['date'][i])==11:
            df['date'][i]=df['date'][i][:5]+'/23'
        else: 
            df['date'][i]=df['date'][i][3:8]+'/'+df['date'][i][:2]
    df['date'] = pd.to_datetime(df['date'])
    return df


df_csv = date_preprocess(make_df())
df_csv.to_csv('everytime_crawling.csv') # csv 파일로 저장
```

<br>

