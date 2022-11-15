---
title: "Github로 블로그 만들기 5편 - 카테고리"
excerpt: "github 블로그의 카테고리를 만들어 게시글들을 분류해보자"

toc: true
toc_label: "목차"

published: true

categories:
  - Githubio

date: 2022-11-15
last_modified_at: 2022-11-15
---
<br>

# Github로 블로그 만들기 5편 - 카테고리

<br>

흔히 우리가 생각하는 블로그의 모습 중에 가장 당연한 것은 카테고리 사이드바다. 카테고리가 없이 분류되지 않은 글은 블로그를 난잡하게 만들고, 원하는 글을 찾는 데에도 어려움을 준다.

<br>

minimal-mistake에는 카테고리가 기본적으로 설정되어 있지 않다. 그래서 `&#95;pages` 폴더에 `categories`라는 폴더를 생설할 것이고 이 안에 직접 카테고리를 md파일로 만들어 넣어줄 것이다.

<br>

```md
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
- **author_profile**: 이 페이지에서 사용자의 프로필이 보이게 할 것인지를 묻는다. 

<br>



