---
title: "Github로 블로그 만들기 5편 - 카테고리"
excerpt: "github 블로그의 카테고리를 만들어 게시글들을 분류해보자"

toc: true
toc_label: "목차"
toc_sticky: true

tags: [blog, github.io]

published: true

categories:
  - Github-io

date: 2022-11-16 00:15:30 +09:00
last_modified_at: 2022-11-16 00:15:30 +09:00
---
<br>

# Github로 블로그 만들기 5편 - 카테고리

<br>

흔히 우리가 생각하는 블로그의 모습 중에 가장 당연한 것은 카테고리 사이드바다. 카테고리가 없이 분류되지 않은 글은 블로그를 난잡하게 만들고, 원하는 글을 찾는 데에도 어려움을 준다.

<br>

## 1. categories 폴더에 카테고리 목록을 md파일로 만들기

minimal-mistake에는 카테고리가 기본적으로 설정되어 있지 않다. 그래서 `&#95;pages` 폴더에 `categories`라는 폴더를 생설할 것이고 이 안에 직접 카테고리를 md파일로 만들어 넣어줄 것이다.

<br>

```
---
title: "Python"
layout: archive
permalink: categories/python
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.Cpp %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
```

<br>

- **title**: 사이드바에 표시될 카테고리의 이름이다. <br>
- **layout**: 카테고리를 만드는 것이므로 이 페이지는 글들을 여러개 모아주는 `archive` 방식으로 설정한다. <br>
- **parmalink**: 페이지의 상대주소를 나타낸다. 이제 게시글을 작성할 때 머리말에서 `categories` 변수에 `python`을 입력하면 그 게시글은 python 카테고리로 배정받는다. <br>
- **author_profile**: 이 페이지에서 사용자의 프로필이 보이게 할 것인지를 묻는다. <br>
- **sidebar_main**: 뒤에 사용할 변수이다. 이 값이 true가 되어야 밑에 나오는 `sidebar.html`이 정상 작동하게 되어 카테고리 사이드바가 나타난다.

<br>

나는 아직 블로그에 게시글 수가 많지 않아 마땅히 카테고리화 시키기에 부족했다. 일단 python, MySQL, github.io 등 대강 카테고리화 시킬 것을 떠올려 문서를 몇 개 만들었다. <br>
<p align='center'>
<img src="https://user-images.githubusercontent.com/115082062/202096184-dd4a898b-7b6a-4eab-be82-542a5ad2b7f1.JPG">
</p>
<br><br>

## 2. nav_list_main 파일에 사이드바를 만들 준비하기

<br>

이렇게 카테고리가 만들어지면 게시글에서 카테고리를 설정할 수는 있지만, 우리가 원하는 것은 홈화면에 띄워지는 사이드바이다. 사이드바를 만들기 위해선 두 가지 문서를 더 작성해야 한다.

<br>

이 부분부터는 [ansohxxn님의 코드](https://ansohxxn.github.io/blog/category/)를 참조하여 작성하였다.

<br>

`github.io` -> `&#95;include` -> `nav_list_main` 파일을 만든다. 이 때 파일 확장자명은 아무것도 적지 않는다. 이 파일에 적은 내용은 다음 단계에서 `sidebar.html`을 작성하 때 쓰일 것이다.

<br>

```html
{% assign sum = site.posts | size %}

{% raw %}
<nav class="nav__list">
  <input id="ac-toc" name="accordion-toc" type="checkbox" />
  <label for="ac-toc">{{ site.data.ui-text[site.locale].menu_label }}</label>
  <ul class="nav__items" id="category_tag_menu">
      <!--전체 글 수-->
      <li>
            📂 <span style="font-family:'Iropke Batang';">전체 글 수</style> <span style="font-family:'Iropke Batang';">{{sum}}</style> <span style="font-family:'Iropke Batang';">개</style> 
      </li>
      <li>
        <!--span 태그로 카테고리들을 크게 분류 ex) 나의 경우 MySQL 관련 내용을 담기 위해 Databse라는 대분류 만들었다-->
        <span class="nav__sub-title">Database</span>
            <!--ul 태그로 같은 카테고리들 모아둔 페이지들 나열-->
            <ul>
                <!--MySQL 카테고리 글들을 모아둔 페이지인 /categories/MySQL 주소의 글로 링크 연결-->
                <!--category[1].size 로 해당 카테고리를 가진 글의 개수 표시--> 
                {% for category in site.categories %}
                    {% if category[0] == "MySQL" %}
                        <li><a href="/categories/MySQL" class="">MySQL ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
            <ul>
                {% for category in site.categories %}
                    {% if category[0] == "programmersMySQL" %}
                        <li><a href="/categories/programmersMySQL" class="">programmersMySQL ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
        <span class="nav__sub-title">Python</span>
            <ul>
                {% for category in site.categories %}
                    {% if category[0] == "Python" %}
                        <li><a href="/categories/Python" class="">Python ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
        <span class="nav__sub-title">Machine Learning</span>
            <ul>
                {% for category in site.categories %}
                    {% if category[0] == "ML" %}
                        <li><a href="/categories/ML" class="">Machine Learning ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
        <span class="nav__sub-title">Github</span>
            <ul>
                {% for category in site.categories %}
                    {% if category[0] == "Githubio" %}
                        <li><a href="/categories/Githubio" class="">Github.io ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
      </li>
  </ul>
</nav>
{% endraw %}
```
<br>

위 내용을 하나씩 뜯어보면,
<br>
```html
{% raw %}
 <!--전체 글 수-->
      <li>
            📂 <span style="font-family:'Iropke Batang';">전체 글 수</style> <span style="font-family:'Iropke Batang';">{{sum}}</style> <span style="font-family:'Iropke Batang';">개</style> 
{% endraw %}
```
<br>
<img src = "https://user-images.githubusercontent.com/115082062/202098405-61661515-0c53-4cc4-a089-d6d2bd1365c9.JPG">

<br>

이 부분은 전체 글 수를 count하는 역할을 해준다. `font-family` 부분에서 폰트를 자유롭게 설정할 수 있다.

<br><br>

```html
        <!--span 태그로 카테고리들을 크게 분류 ex) 나의 경우 MySQL 관련 내용을 담기 위해 Databse라는 대분류 만들었다-->
        <span class="nav__sub-title">Database</span>
            <!--ul 태그로 같은 카테고리들 모아둔 페이지들 나열-->
            <ul>
```

<br>
<img src = "https://user-images.githubusercontent.com/115082062/202099326-9067a147-a03e-49da-9946-f5691e66dd23.JPG">

<br>

`class="nav__sub-title"`은 소분류를 묶는 대분류를 나타내준다. 위 경우엔 Database가 대분류이다.

<br><br>

```html
{% raw %}
<ul>
                <!--MySQL 카테고리 글들을 모아둔 페이지인 /categories/MySQL 주소의 글로 링크 연결-->
                <!--category[1].size 로 해당 카테고리를 가진 글의 개수 표시--> 
                {% for category in site.categories %}
                    {% if category[0] == "MySQL" %}
                        <li><a href="/categories/MySQL" class="">MySQL ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
            <ul>
                {% for category in site.categories %}
                    {% if category[0] == "programmersMySQL" %}
                        <li><a href="/categories/programmersMySQL" class="">programmersMySQL ({{category[1].size}})</a></li>
                    {% endif %}
                {% endfor %}
            </ul>
{% endraw %}
```

<br>

<img src="https://user-images.githubusercontent.com/115082062/202099821-ae4ff117-868c-4fde-99c0-1f5565c90c2d.JPG">

<br>

이제는 소분류가 등장한다. 나는 MySQL과 programmersMySQL이란 두 개의 소분류를 만들었는데, 캡쳐본에 programmersMySQL밖에 안 나타나는 이유는 내가 아직 MySQL 카테고리의 글을 한 개도 안 올렸기 때문이다.....

<br><br>

## 3. sidebar.html 파일에 내용 추가하기
<br>

이제 `github.io` -> `&#95;include` -> `sidebar.html` 파일로 이동해서 코드 세 줄을 추가해 줄 것이다.

```html
{% raw %}
{% if page.author_profile or layout.author_profile or page.sidebar %}
  <div class="sidebar sticky">
  {% if page.author_profile or layout.author_profile %}{% include author-profile.html %}{% endif %}
  {% if page.sidebar %}
    {% for s in page.sidebar %}
      {% if s.image %}
        <img src="{{ s.image | relative_url }}"
             alt="{% if s.image_alt %}{{ s.image_alt }}{% endif %}">
      {% endif %}
      {% if s.title %}<h3>{{ s.title }}</h3>{% endif %}
      {% if s.text %}{{ s.text | markdownify }}{% endif %}
      {% if s.nav %}{% include nav_list nav=s.nav %}{% endif %}
    {% endfor %}
    {% if page.sidebar.nav %}
      {% include nav_list nav=page.sidebar.nav %}
    {% endif %}
  {% endif %}
  
  {% if page.sidebar_main %} <!--추가할 부분-->
    {% include nav_list_main %} <!--추가할 부분-->
  {% endif %} <!--추가할 부분-->

  </div>
{% endif %}
{% endraw %}
```

<br>

위에 주석표시한 세 줄이 추가할 코드이다.

<br><br>

## 4. &#95;config.yml 수정

<br>

이제 정말 다 왔다. `&#95;config.yml` 파일에 `sidebar_main: true`을 추가해주기만 하면 사이드바가 활성화된다!

<br>

```yml
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
      sidebar_main: true #이부분 추가!
```

<br>