---
title: "Github로 블로그 만들기 2편 - 포스팅하기"
excerpt: "github 블로그에 첫 게시글을 업로드해보자"

toc: true
toc_label: "목차"

published: true

date: 2022-11-15
last_modified_at: 2022-11-15
---
<br>


# Github로 블로그 만들기 2편 - 포스팅하기

<br>
이제 대략적인 기본 설정을 마쳤으니, 본격적인 첫 포스팅을 진행해보자. 우선 포스팅을 하기 위한 폴더를 생성해주어야 한다.
<br><br>
github.io 폴더 -> &#95;posts 폴더 생성!
<br><br>
그런 다음 md 파일을 하나 생성해주는데, 이 때 제목은 규격을 무조건 따라야 한다. yyyy-mm-dd-원하는제목.md로 만들어주면 된다.
<br><br>
글의 가장 처음 와야 할 것은 아래와 같은 Front-Matter이다. 모든 내용을 다 적을 필요는 없고, 기본적으로 title 정도는 무조건 적어주어야 한다. <br><br>
나의 경우 처음 글을 포스팅할 때 몇 번을 시도해봐도 업로드가 안 되었는데, published: true를 입력해주고 나서 해결됐다.
<br><br>

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

<br><br>

<img src= "https://user-images.githubusercontent.com/115082062/201882407-2cff03e8-0208-42e6-b8e4-688498b9e5e2.png">
<br>
excerpt에 작성한 내용이 제목 밑에 뜨는 것을 볼 수 있다.

<br><br>

<img src= "https://user-images.githubusercontent.com/115082062/201882798-dce6c216-3d9d-48d4-9677-2e8b4d630dee.png">
<br>
글이 잘 업로드 되었다. 오른쪽에 목차가 생성된 것도 확인할 수 있다.
<br>