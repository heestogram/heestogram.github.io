---
title: "Github로 블로그 만들기 4편 - 폰트 변경"
excerpt: "github 블로그 폰트를 변경해보자"

toc: true
toc_label: "목차"

published: true

categories:
  -githubio

date: 2022-11-15
last_modified_at: 2022-11-15
---

<br>

# Github로 블로그 만들기 4편 - 폰트 변경

<br>

## 1. 맘에 드는 폰트를 웹에서 찾기

<br>

Github 블로그에서 폰트를 바꾸기 위해선 다운로드 받아 로컬에 저장하는 형태가 아니라, **웹폰트의 url을 css에 import**하는 방법을 사용해야 한다. <br><br>

우선 원하는 폰트를 골라야 한다. 무료 폰트 사이트인 [눈누](https://noonnu.cc/)를 이용하기로 했다. <br><br>

원하는 폰트를 고르고 '웹폰트로 사용' 부분을 복사해준다. <br>

<img src= "https://user-images.githubusercontent.com/115082062/201917461-4e10344a-4e9b-4a81-b4ee-ac40f31de991.JPG">

<br><br>

아래 내용을 복사하면 된다.
<br>

```scss
@font-face {
    font-family: 'LeferiPoint-WhiteObliqueA';
    src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2201-2@1.0/LeferiPoint-WhiteObliqueA.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}
```
<br><br>

## 2. main.scss에 폰트 정보 import하기

<br>

그 후에 `assets` 폴더 -> `css` 폴더 -> `main.scss`로 이동하여 복사한 내용을 그대로 붙여넣어준다. <br>

```scss
@charset "utf-8";

@import "minimal-mistakes/skins/{{ site.minimal_mistakes_skin | default: 'default' }}"; // skin
@import "minimal-mistakes"; // main partials
@import url('https://cdn.jsdelivr.net/font-iropke-batang/1.2/font-iropke-batang.css');
@font-face {
    font-family: 'LeferiPoint-WhiteObliqueA';
    src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2201-2@1.0/LeferiPoint-WhiteObliqueA.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}
```

<br>

이 때 `font-family`에 적힌 부분을 복사한다. 위의 경우는 'LeferiPoint-WhiteObliqueA'이다. 이 부분을 폰트의 이름으로 생각하면 된다.

<br><br> 

## 3. &#95;variables.scss에 폰트를 변수로 할당하기
<br>

폰트의 이름은 `&#95;variables.scss`에 변수로 할당해놓을 수 있다.

<br>

```scss
/* system typefaces */
$serif: Georgia, Times, serif !default;
$sans-serif: -apple-system, BlinkMacSystemFont, "Roboto", "Segoe UI",
  "Helvetica Neue", "Lucida Grande", Arial, sans-serif !default;
$monospace: Monaco, Consolas, "Lucida Console", monospace !default;
$Lefer: "LeferiPoint-WhiteObliqueA" !default;
```

<br>

여러 가지 폰트가 `$변수명`의 방식으로 변수로 할당되어 있다. 나는 `$Lefer`라는 새로운 변수명을 만들고 그 옆에 아까 복사해두었던 폰트명을 붙여넣어주었다.<br><br>

스크롤을 조금 내리면 아래와 같은 부분이 보인다. 폰트를 변경하고 싶은 부분에 아까 새로 설정한 변수를 넣어주면 된다. <br>

```scss
$global-font-family: $Lefer !default;
$header-font-family: $Lefer !default;
$caption-font-family: $Lefer !default;
```

<br>

여기까지 하면 폰트가 깔끔하게 바뀐다!
<br>