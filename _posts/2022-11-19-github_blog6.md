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

## &#95;layouts 폴더 -> `single.html`

&#95;layouts 폴더에는 각종 문서들이 블로그에서 어떻게 표현될지를 나타내는 문서들의 모임이다. 우리가 게시글을 포스팅할 때 머리말의 `layout`은 default값이 `single`이다. 때문에 &#95;layouts 폴더의 `single.html` 파일을 건드려주는 것이다.

```html
    <div class="page__inner-wrap">
      {% unless page.header.overlay_color or page.header.overlay_image %}
        <header>
          {% if page.title %}<h1 id="page-title" class="page__title p-name" itemprop="headline">
            <a href="{{ page.url | absolute_url }}" class="u-url" itemprop="url">{{ page.title | markdownify | remove: "<p>" | remove: "</p>" }}</a>
          </h1>{% endif %}
          <!--{% include page__meta.html %}-->
          {% if page.last_modified_at and page.date %}
            <p class="page__meta"><strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Date:" }}</strong> <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y.%m.%d" }}</time>&nbsp&nbsp&nbsp<strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Updated:" }}</strong> <time datetime="{{ page.last_modified_at | date: date_to_xmlschema }}">{{ page.last_modified_at | date: "%Y.%m.%d" }}</time></p>
          {% elsif page.date %}
            <p class="page__meta"><strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Date:" }}</strong> <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y.%m.%d" }}</time></p>
          {% elsif page.last_modified_at %}
            <p class="page__meta"><strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Updated:" }}</strong> <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y.%m.%d" }}</time></p>
          {% endif %}
          {% if site.category_archive.type and page.categories[0] and site.tag_archive.type and page.tags[0] %}
            {% include category-list.html %}{% include tag-list.html %}
          {% elsif site.category_archive.type and page.categories[0] %}
            {% include category-list.html %}
          {% elsif site.tag_archive.type and page.tags[0] %}
            {% include tag-list.html %} 
          {% endif %}
        </header>
      {% endunless %}

```
`class="page__inner-wrap"`부분을 보면 header 정보가 나와있다. 위 코드에서 주석처리 된 부분은 소요시간을 나타내는 부분인데, 나는 소요시간을 빼고 싶어서 주석처리를 했다.

<br>

## 게시날짜, 수정날짜 추가하기
위 코드에서 게시날짜와 수정날짜를 추가하는 코드를 발췌했다. 우리가 글을 포스팅할 때 머리말에서 `last_modified_at`과 `date` 요소를 설정해주는 것을 기억할 테다. 아래 코드는 그 요소들이 존재한다면 해당 아이콘과 함께 날짜를 출력해주는 if 조건문이다.

```html
          {% if page.last_modified_at and page.date %}
            <p class="page__meta"><strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Date:" }}</strong> <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y.%m.%d" }}</time>&nbsp&nbsp&nbsp<strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Updated:" }}</strong> <time datetime="{{ page.last_modified_at | date: date_to_xmlschema }}">{{ page.last_modified_at | date: "%Y.%m.%d" }}</time></p>
          {% elsif page.date %}
            <p class="page__meta"><strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Date:" }}</strong> <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y.%m.%d" }}</time></p>
          {% elsif page.last_modified_at %}
            <p class="page__meta"><strong><i class="fas fa-fw fa-calendar-alt" aria-hidden="true"></i> {{ "Updated:" }}</strong> <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y.%m.%d" }}</time></p>
          {% endif %}
```


<br>

## 카테고리, 태그 추가하기
위 코드에서 카테고리와 태그를 추가하는 코드를 발췌했다. 우리가 글을 포스팅할 때 머리말에서 `categories`과 `tag` 요소를 설정해주는 것을 기억할 테다. 아래 코드는 그 요소들이 존재한다면 해당 아이콘과 함께 날짜를 출력해주는 if 조건문이다.

```html
          {% raw %}
          {% if site.category_archive.type and page.categories[0] and site.tag_archive.type and page.tags[0] %}
            {% include category-list.html %}{% include tag-list.html %}
          {% elsif site.category_archive.type and page.categories[0] %}
            {% include category-list.html %}
          {% elsif site.tag_archive.type and page.tags[0] %}
            {% include tag-list.html %} 
          {% endif %}
          {% endraw %}
```

<img src="https://user-images.githubusercontent.com/115082062/202852888-47c79e61-6388-497d-996b-52864f8387b2.JPG">

코드를 모두 추가해줬다면 위 사진처럼 바뀐다! 아주 마음에 든다.

<br>

혹시나 코드블럭이 이상하게 보인다면, 내 블로그 깃허브 소스트리를 방문해서 코드를 복붙해주면 된다! [heestogram single.html 링크](https://github.com/heestogram/heestogram.github.io/blob/master/_layouts/single.html)
