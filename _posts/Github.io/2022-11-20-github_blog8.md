---
title: "Github로 블로그 만들기 8편 sidebar와 masthead 폰트 크기 수정"
excerpt: "github 블로그 sidebar의 프로필부분과 masthead의 폰트 크기를 수정해보자"

toc: true
toc_label: "목차"
toc_sticky: true

tags: [blog, github.io, Jekyll, minimal-mistakes]

published: true

categories:
  - Github-io

date: 2022-11-19 23:40:30
last_modified_at: 2022-11-20 00:51:30
---

<br>

블로그 상단의 제목이나 프로필 이름을 보면 좀 작다는 생각이 든다. 시원시원하게끔 폰트 크기를 키우고 폰트 종류도 변경해보려고 한다.

<br>

## masthead 수정하기
블로그의 가장 상단에 위치한 타이틀이 masthead이다. 나는 이 부분의 폰트 크기를 키워보려고 한다. 그러기 위해선 `_masthead.scss` 파일로 이동해야 한다.

```scss
.site-title {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-item-align: center;
  align-self: center;
  font-size: $type-size-4; /*이 부분 추가*/
  font-weight: bold;
  // z-index: 20;
}

.site-subtitle {
  display: block;
  font-size: $type-size-8;
}
```
위 코드 부분을 찾아서 원하는대로 커스터마이징해준다.<br> 
site-title이 블로그의 제목, site-subtitle은 제목 밑에 작게 나타나는 부제목이다.

<br>

`$type-size`가 어느정도 크기인지 모르겠다면 `_sass` 폴더 -> `minimal-mistakes` 폴더 -> `_variable.scss` 파일로 가서 아래와 같은 정보를 확인할 수 있다.

```scss
/* type scale */
$type-size-1: 2.441em !default; // ~39.056px
$type-size-2: 1.953em !default; // ~31.248px
$type-size-3: 1.563em !default; // ~25.008px
$type-size-4: 1.25em !default; // ~20px
$type-size-5: 1em !default; // ~16px
$type-size-6: 0.75em !default; // ~12px
$type-size-7: 0.6875em !default; // ~11px
$type-size-8: 0.625em !default; // ~10px
```

<br>

## sidebar 프로필 수정하기
블로그 왼편에 나타나는 프로필을 수정하고 싶다면 `_sass` 폴더 -> `minimal-mistakes` 폴더 -> `_sidebar.scss` 파일로 가자.

```scss
.sidebar .author__name {
  font-family: $Iropke;
  font-size: $type-size-4;
}

.author__bio {
  margin: 0;
  font-family: $Iropke;

  @include breakpoint($large) {
    margin-top: 10px;
    margin-bottom: 20px;
  }
}

.author__urls-wrapper {
  position: relative;
  display: table-cell;
  vertical-align: middle;
  font-family: $sans-serif;
  z-index: 20;
  cursor: pointer;
}
```

`sidebar .author__name`이 프로필 사진 밑에 적히는 이름이고, `author__bio`가 그 밑에 적히는 소개글이다. `author__urls-wrapper`에서는 그 밑에 나오는 개인사이트 링크들을 손볼 수 있다. <br>
`font-family`에는 원하는 폰트를 변수로 설정할 수 있고, `font-size`로 폰트의 크기를 조절할 수 있다.

<br>

<img src="https://user-images.githubusercontent.com/115082062/202867583-2c18f049-dd36-4841-b04f-3bcbcba32e85.JPG">

바뀐 사이드바의 프로필과 masthead의 모습이다. 만족!