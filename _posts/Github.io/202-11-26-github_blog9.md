---
title: "Github로 블로그 만들기 9편 - inline code 색상 변경"
excerpt: "github 블로그의 inline 코드 색상을 눈에 더 잘 띄게 바꿔 보자"

toc: true
toc_label: "목차"
toc_sticky: true

tags: [blog, github.io, Jekyll, minimal-mistakes]

published: true

categories:
  - Github-io

date: 2022-11-26 01:56:30
last_modified_at: 2022-11-26 01:56:30
---

<br>

minimal mistakes의 기본 테마에서는 inline 코드의 배경색과 폰트색이 눈에 너무나도 안 띈다. 사소한 부분인 것 같으면서도, 인라인 코드를 자주 쓰는 사람들에겐 치명적인 단점일 수 있다.

<br>

그럼 인라인 코드의 배경색과 폰트색을 바꿔보자. 방법은 정말 간단하다.
<br>

- `_sass` -> `minimal-mistakes` -> `_base.scss`로 이동한다.

163열의 아래와 같은 코드를 복사 붙여넣기 해준다.

``` scss
p > code,
a > code,
li > code,
figcaption > code,
td > code {
  padding-top: 0.1rem;
  padding-bottom: 0.1rem;
  font-size: 0.8em;
  background: $code-background-color;
  border-radius: $border-radius;
  color: #eb6060;
  &:before,
  &:after {
    letter-spacing: -0.2em;
    content: "\00a0"; /* non-breaking space*/
  }
}
```

여기서 color로 주어진 #eb6060가 바로 폰트색이다. 그리고 코드의 배경색은 background에 입력된 $code-background-color라는 색상 변수인데, 이 색깔이 무엇인지는 같은 디렉토리의 `_variables.scss`에서 찾을 수 있다.

```scss
   Colors
   ========================================================================== */

$gray: #7a8288 !default;
$dark-gray: mix(#000, $gray, 50%) !default;
$darker-gray: mix(#000, $gray, 60%) !default;
$light-gray: mix(#fff, $gray, 50%) !default;
$lighter-gray: mix(#fff, $gray, 90%) !default;

$background-color: #fff !default;
$code-background-color: #e9dcbe !default; //#fafafa
$code-background-color-dark: $light-gray !default;
$text-color: $dark-gray !default;
$muted-text-color: mix(#fff, $text-color, 20%) !default;
$border-color: $lighter-gray !default;
$form-background-color: $lighter-gray !default;
$footer-background-color: $lighter-gray !default;
```
바로 저 부분의 $code-background-color: #e9dcbe이 바로 인라인 코드의 배경색을 지정해주는 부분이다! 따라서 `_variables.scss`파일의 저 부분의 색상을 다른 값으로 주면 자동으로 `_base.scss`파일에 반영되어 배경색이 바뀔 것이다.

<br>

`지금 보고 있는 이 인라인 코드가 바로 수정된 결과물이다` 더 최신화된 내용은 내 깃허브 링크를 참고하면 좋다.
<br>

- [`_variables.scss` 파일 링크](https://github.com/heestogram/heestogram.github.io/blob/master/_sass/minimal-mistakes/_variables.scss)
- [`_base.scss` 파일 링크](https://github.com/heestogram/heestogram.github.io/blob/master/_sass/minimal-mistakes/_base.scss)
