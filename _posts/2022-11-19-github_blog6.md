---
title: "Github로 블로그 만들기 6편 - 본문의 제목 form 수정"
excerpt: "github 블로그의 제목 부분에 게시날짜, 카테고리, 태그를 추가해보자"

toc: true
toc_label: "목차"
toc_sticky: true

tags: [blog, github.io, Jekyll, minimal-mistakes]

published: true

categories:
  - Github-io

date: 2022-11-19 21:54:30
last_modified_at: 2022-11-16 21:54:30
---
<br>

<img src = "https://user-images.githubusercontent.com/115082062/202851909-da34005a-c1c0-44b0-87ae-f11407620d10.JPG">

minimal-mistakes 테마는 보다시피 제목 부분에 제목과 소요시간만 덩그러니 제공된다. 이 부분에 해당 게시글의 게시날짜, 수정날짜, 카테고리, 태그를 만들어보고자 한다.

<br>

**&#95;layouts 폴더 -> `single.html`**

&#95;layouts 폴더에는 각종 문서들이 블로그에서 어떻게 표현될지를 나타내는 문서들의 모임이다. 우리가 게시글을 포스팅할 때 머리말의 `layout`은 default값이 `single`이다. 때문에 &#95;layouts 폴더의 `single.html` 파일을 건드려주는 것이다.

