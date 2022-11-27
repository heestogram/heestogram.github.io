---
title: "Github로 블로그 만들기 1편"
excerpt: "github 블로그를 생성한 후 해야 할 가장 간단한 기본설정을 알아보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

tags: [blog, github.io, Jekyll, minimal-mistakes]

categories:
  - Github-io

date: 2022-11-15
last_modified_at: 2022-11-15
---
<br>

<div class='notice--primary' markdown='1'>
💡
Jekyll에서 제공되는 minimal-mistakes 테마를 기본으로 하여 커스터마이징 해가는 과정이다.
</div>


## 1. 기본 정보 변경
<br>
github.io 폴더 -> &#95;config.yml
<br>
<br>
가장 기본적인 초기 정보 setting이다.
<br>

```yml
# Site Settings
locale                   : "ko-KR" #사용 언어
title                    : "Heestogram's Heestudy" #블로그 이름
title_separator          : "-"
subtitle                 : "Catch Jun If U Can" #블로그 이름 밑에 들어갈 부제목
name                     : "Hee Jun Kim"  #사용자 이름
description              : "study portfolio" #블로그에 대한 설명
url                      : "https://heestogram.github.io" #블로그 주소
baseurl                  : # the subpath of your site, e.g. "/blog"
repository               : "https://github.com/heestogram/heestogram.github.io" #해당 레퍼지터리 주소
teaser                   : # path of fallback teaser image, e.g. "/assets/images/500x300.png"
logo                     : # path of logo image to display in the masthead, e.g. "/assets/images/88x88.png"
masthead_title           : "Heestogram's Heestudy" #블로그 이름
# breadcrumbs            : false # true, false (default)
words_per_minute         : 200
```
<br>

스크롤을 조금 더 내리면 아래와 같은 내용이 있다. Author, 즉 블로그 사용자의 정보를 수정하는 곳이다.<br>
블로그 홈에 표시될 사용자 정보에 나타나는 내용이라고 이해하면 쉽다.
강조하고 싶은 부분은 &#42;&#42;로 감싸자.
<br><br>
또한 links 부분을 수정하여 블로그 좌단에 링크를 걸 수도 있다. 주석처리하면 그 링크는 표시되지 않는다.
<br>
email의 url을 설정할 때는 다른 것과 달리 앞에 mailto: 를 붙여줘야 한다.
<br>
```yml
# Site Author
author:
  name             : "Hee Jun Kim"
  avatar           : # path of avatar image, e.g. "/assets/images/bio-photo.jpg"
  bio              : "Wanna be a **Data Analyst**" #사용자 이름 밑에 들어가는 bio
  location         : "Seoul, Republic of Korea" #사용자 출신지
  email            :
  links:
    - label: "anthjoon11@naver.com" #블로그에 표시될 링크의 제목
      icon: "fas fa-fw fa-envelope-square" #설정된 아이콘값. 각각 고유의 값이 있음
      url: mailto:anthjoon11@naver.com #이 부분만 주석 처리하면 블로그에 표시 안 할 수 있음
    - label: "Website"
      icon: "fas fa-fw fa-website"
      # url: "https://github.com/heestogram"
    - label: "Twitter"
      icon: "fab fa-fw fa-twitter-square"
      # url: "https://twitter.com/"
    - label: "Facebook"
      icon: "fab fa-fw fa-facebook-square"
      # url: "https://facebook.com/"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/heestogram"
    - label: "Instagram"
      icon: "fab fa-fw fa-instagram"
      # url: "https://instagram.com/"
 ```
 <br>
 초기설정이 끝났다면 아래와 같은 화면이 생성된다. 좌단에는 프로필 소개가 잘 들어가 있고, 하단에는 footer에서 기입한 내용(github icon)이 보일 것이다.
 <br><br>
 캡쳐 화면에 보이는 최근 포스트 업로드 방법과 우측상단 카테고리들 만드는 법은 다음 단계에서 차차 확인할 수 있다.
 <br><br>
 <img src= "https://user-images.githubusercontent.com/115082062/201826923-2a2482e8-c8a6-4955-868a-651f4d48e3d7.JPG">
<br>

## 2. 상단 메뉴바 만들기

<br>
 github.io 폴더 -> &#95;data 폴더 -> navigation.yaml 파일
<br><br>
 무척이나 간단하다. 생성하고 싶은 메뉴의 title을 적어주고 url을 설정해주면 된다. 이 때 주석처리는 당연히 풀어줘야 한다.<br><br>
 아래처럼 작성하면 "About Me"라는 타이틀에 /me/라는 url을 걸어놓은 것이 되는데, &#95;pages에 들어가서 md파일을 하나 생성한 후 permalink를 /me/로 수정해주면 "About Me' 메뉴를 눌렀을 때 해당 md파일로 링크가 되는 것이다.
<br><br>
```yml
# main links
main:
  - title: "About Me"
    url: /me/
  # - title: "About"
  #   url: https://mmistakes.github.io/minimal-mistakes/about/
  # - title: "Sample Posts"
  #   url: /year-archive/  #안 보이게 하고 싶은 부분은 주석처리
  # - title: "Sample Collections"
  #   url: /collection-archive/
  # - title: "Sitemap"
  #   url: /sitemap/
```
<br><br>
github.io 폴더 -> &#95;pages 폴더 <br><br>
이곳에서 about-me.md라는 파일 하나를 만들고 아래와 같이 작성해 보겠다.
<br>
```yml
---
permalink: /me/
title: "About Me"
toc: true
---
반갑습니다.
```
<br>
상단메뉴 'About Me'를 클릭하면 아래와 같은 창이 열리는 것을 확인할 수 있다.
<br><br>
<img src= "https://user-images.githubusercontent.com/115082062/201839699-fc8704c9-4ef3-47ac-a955-1b79eb713690.JPG">
<br>
<br>

## 3. 너비와 폰트 크기 수정
<br>

github.io 폴더 -> &#95;sass 폴더 -> &#95;minimal-mistakes 폴더 -> &#95;variables.scss
<br><br>
&#95;variables.scss 파일의 가장 밑쪽으로 가서 너비를 알맞게 수정해준다.<br>

```scss
   Grid
   ========================================================================== */

$right-sidebar-width-narrow: 100px !default;
$right-sidebar-width: 200px !default;
$right-sidebar-width-wide: 250px !default;
```
<br>
minimal-mistakes의 default 폰트 크기는 꽤 커서 읽기에 불편함이 있다. 폰트 사이즈를 줄여보도록 하자.
<br><br>
github.io 폴더 -> &#95;sass 폴더 -> &#95;minimal-mistakes 폴더 -> &#95;reset.scss
<br><br>

```scss
html {
  /* apply a natural box layout model to all elements */
  box-sizing: border-box;
  background-color: $background-color;
  font-size: 14px; # 이 부분 수정

  @include breakpoint($medium) {
    font-size: 16px; # 이 부분 수정
  }

  @include breakpoint($large) {
    font-size: 16px; # 이 부분 수정
  }

  @include breakpoint($x-large) {
    font-size: 16px; # 이 부분 수정
  }

  -webkit-text-size-adjust: 100%;
  -ms-text-size-adjust: 100%;
}
```
<br>

## 4. 파비콘 추가하기

<br>
웹사이트 상단 바에 표시되는 아이콘을 파비콘이라고 한다. 파비콘을 만들기 위해 이 사이트에 접속한다. https://realfavicongenerator.net/
<br><br>
아래 웹사이트에서 'select your favicon image'를 클릭한다. 그리고 내가 원하는 이미지를 갖고온다.
<br><br>
<img src = "https://user-images.githubusercontent.com/115082062/201844820-440d43d5-6b32-480d-80e5-e97dbeb2f82d.JPG">
<br><br>
그럼 아래와 같이 내가 만든 파비콘의 코드가 나올 것이다. 우선 1. Download your package에서 파비콘 이미지 파일들을 다운 받는다.
<br><br>
github.io 폴더 -> assets 폴더 -> logo.ico 폴더를 생성하여 이곳에 파비콘 이미지 파일들을 붙여넣는다.
<br><br>
그리곤 제공된 코드를 복사하자.
<br><br>
<img src = "https://user-images.githubusercontent.com/115082062/201845237-ee469c41-3a37-4f73-8cd4-49713f559844.JPG">
<br><br>
github.io 폴더 -> &#95;includes 폴더 -> &#95;head 폴더 -> custom.html
<br><br>
복사한 코드를 insert하라는 부분 바로 밑에 붙여넣어주면 된다. 이 때 아래 코드처럼 href에 {{site.baseurl}}/assets/logo.ico 을 추가해주어야 한다.
<br><br>

```html
<!-- start custom head snippets -->

<!-- insert favicons. use https://realfavicongenerator.net/ -->
<link rel="apple-touch-icon" sizes="60x60" href="{{site.baseurl}}/assets/logo.ico/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="{{site.baseurl}}/assets/logo.ico/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="{{site.baseurl}}/assets/logo.ico/favicon-16x16.png">
<link rel="manifest" href="{{site.baseurl}}/assets/logo.ico/site.webmanifest">
<link rel="mask-icon" href="{{site.baseurl}}/assets/logo.ico/safari-pinned-tab.svg" color="#5bbad5">
<meta name="msapplication-TileColor" content="#da532c">
<meta name="theme-color" content="#ffffff">

<!-- end custom head snippets -->
```

<br>
<p align='center'>
  <img src = "https://user-images.githubusercontent.com/115082062/201922633-cfac2d02-b7c6-4581-b527-e9bef3c41fab.JPG">
</p>

한참동안 파비콘이 업데이트가 안되길래, 코드를 잘못 적은 줄 알았다. ~~요망한 네잎클로버 녀석 드디어 뜬다.~~


