---
title: "Github로 블로그 만들기 3편 - 구체적인 개인설정"
excerpt: "github 블로그에 프로필 사진, 카테고리 등을 만들어보자"

toc: true
toc_label: "목차"

published: true

date: 2022-11-15
last_modified_at: 2022-11-15
---

<br>

# Github로 블로그 만들기 3편 - 구체적인 개인설정

<br>

## 1. 프로필 사진 추가

<br>

이제 기본적인 뼈대는 완성됐지만, 아직 블로그라 하기엔 심미적으로 부족한 부분이 많다. 이를 해소하기 위해 프로필 사진을 추가해볼 것이다. <br><br>

우선 원하는 사진을 저장하여 assets 폴더에 저장해준다. 파일명은 어떤 것이든 상관 없다. &#95;config.yml로 가서 `avatar` 부분을 찾는다. <br>

```yml
author:
  name             : "Hee Jun Kim"
  avatar           : "/assets/avatar.png"
  bio              : "Wanna be a **Data Analyst**" 
  location         : "Seoul, Republic of Korea" 
  email            :
```
<br>

어렵지 않게 찾을 수 있다. 이 부분에 경로를 "assets/파일명"으로 기입해주면 된다. 나의 경우 사진 파일명이 avatar.png로 저장되었기에 위와 같이 작성했다. <br><br>
<img src = "https://user-images.githubusercontent.com/115082062/201903782-f4c8ed7f-447b-490a-aefe-1c38c4165d33.JPG">

<br>

프로필 사진이 잘 들어간 모습이다.


