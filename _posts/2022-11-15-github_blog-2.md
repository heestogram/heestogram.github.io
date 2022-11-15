---
title: "Github로 블로그 만들기 2편 - 포스팅하기"
excerpt: "github 블로그에 첫 게시글을 업로드해보자"

toc: true
toc_label: "목차"

published: true

categories:
  -githubio

date: 2022-11-15
last_modified_at: 2022-11-15
---
<br>


# Github로 블로그 만들기 2편 - 포스팅하기

<br>

## 1. &#95;posts 폴더 만들기
<br>

이제 대략적인 기본 설정을 마쳤으니, 본격적인 첫 포스팅을 진행해보자. 우선 포스팅을 하기 위한 폴더를 생성해주어야 한다. 이 폴더가 없으면 글을 담아놓을 공간도 없고, 업로드도 되지 않는다.
<br><br>
github.io 폴더 -> &#95;posts 폴더 생성!
<br><br>

## 2. md 파일 생성하기

<br>

이제 md 파일을 하나 생성해줄 것이다. 이 파일이 우리의 게시글이 된다. 이 때 제목은 규칙을 무조건 따라야 한다. **yyyy-mm-dd-원하는제목.md**로 만들어주면 된다. 나는 로컬의 VSC로 글을 작성하는데, 제목을 규칙에 엇나가게 지으면 not valid하다며 경고를 한다.<br><br>

확장자명이 md라는 것은 markdown 문법을 따른다는 것이다. 다른 언어에 비해 비교적 간편하게 작성할 수 있다는 장점이 있다. 기본적인 syntax가 어렵지 않으니 기초를 익히고 글을 많이 많이 써보며 익숙해져 보려고 한다!<br><br>

## 3. Front-Matter(머릿말) 작성하기

<br>

글의 가장 처음 와야 할 것은 아래와 같은 **Front-Matter**이다. 모든 내용을 다 적을 필요는 없고, 기본적으로 title 정도는 무조건 적어주어야 한다. <br><br>


```md
---
title: "Github로 블로그 만들기 1편"
excerpt: "github 블로그를 생성한 후 해야 할 가장 간단한 기본설정을 알아보자"

toc: true
toc_label: "목차"

published: true

date: 2022-11-15
last_modified_at: 2022-11-15
---
```

<br>
Front-Matter에서 자주 쓰이는 내용들을 테이블로 정리했다.
<br><br>

|name|mean|
|---|---|
|title|실제 화면에 보이는 제목|
|excerpt|게시글 제목 밑에 보이는 간략한 글 소개|
|categories|게시글 카테고리|
|tags|게시글에 들어갈 태그|
|toc|글의 목차. true로 설정하면 글의 헤딩을 기준으로 목차가 자동생성된다.|
|toc_label|toc의 이름|
|toc_icon|toc의 아이콘|
|toc_sticky|toc의 고정유무, 고정(true)시 스크롤에도 우측상단에 고정되게 보인다.|
|data|게시글 작성일|
|last_modified_at|게시글 마지막 수정일|

<br>

번외로, 나의 경우 처음 글을 포스팅할 때 몇 번을 시도해봐도 업로드가 안 되었는데, 구글링해본 결과 해결방법은 다음과 같았다. <br><br>
1)머릿말에 `published: true`를 입력한다. <br>
2)&#95;config_yaml에서 맨 밑에 `future:true`를 입력한다.<br><br><br>

<img src= "https://user-images.githubusercontent.com/115082062/201882407-2cff03e8-0208-42e6-b8e4-688498b9e5e2.png">
<br>
정상적으로 업로드가 완료됐다면, excerpt에 작성한 내용이 위 사진처럼 제목 밑에 뜨는 것을 볼 수 있다.

<br><br>

<img src= "https://user-images.githubusercontent.com/115082062/201882798-dce6c216-3d9d-48d4-9677-2e8b4d630dee.png">
<br>
글이 잘 업로드 되었다. 오른쪽에 목차가 생성된 것도 확인할 수 있다.
<br>
