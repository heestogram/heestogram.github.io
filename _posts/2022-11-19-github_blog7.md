---
title: "Github로 블로그 만들기 7편 - liquid 언어 코드 포스팅 시 오류 해결"
excerpt: "github 블로그 코드 블럭에 liquid 언어가 잘못 포스팅되는 것을 해결해보자"

toc: true
toc_label: "목차"
toc_sticky: true

tags: [blog, github.io, Jekyll, minimal-mistakes, liquid, error, HTML]

published: true

categories:
  - Github-io

date: 2022-11-19 23:40:30
last_modified_at: 2022-11-19 23:40:30
---

<br>

minimal-mistake 파일들을 만지다보면 html 파일에서 중괄호로 된 `liquid` 언어를 심심찮게 볼 수 있을 것이다.
html은 정적인 언어이다. 때문에 변수나 함수처럼 동적인 처리를 할 수가 없다. 하지만 liquid는 동적인 언어이기 때문에 정적인 html에서 변수와 함수, 논리를 구현할 수 있게 해준다.
당연히 블로그 포스팅할 때도 코드블럭을 사용하면 liquid 언어가 많이 삽입될 수밖에 없다. 그런데 내가 봉착한 문제는 다음과 같다.

<br>

<img src="https://user-images.githubusercontent.com/115082062/202856599-91170031-07c8-4f6d-ab05-4436e97124e3.JPG">

분명 나는 vscode에서 위와 같은 코드블럭을 형성했는데,

<br>

<img src="https://user-images.githubusercontent.com/115082062/202856629-43c399ed-ebfe-40cb-b435-1dd80293b76b.JPG">

실제로 글을 포스팅하고 보니 위처럼 난데없는 코드가 뜨는 것이 아닌가.

이는 liquid의 특성상 저 코드를 그대로 쓰면 예시코드를 출력하려는 특성이 있어서 다른 결과물이 나와버리는 것이다.

<br>

이를 해결하는 방법은 매우 간단하다. 아래에서 주석처리한 부분 `{% raw %}`와 `{% endraw %}`로 감싸주면 해결된다. 당연히 주석은 풀고 감싸주도록 하자.

```html
          <!--{% raw %}-->
          {% raw %}
          {% if site.category_archive.type and page.categories[0] and site.tag_archive.type and page.tags[0] %}
            {% include category-list.html %}{% include tag-list.html %}
          {% elsif site.category_archive.type and page.categories[0] %}
            {% include category-list.html %}
          {% elsif site.tag_archive.type and page.tags[0] %}
            {% include tag-list.html %} 
          {% endif %}
          {% endraw %}
          <!--{% endraw %}-->
```



