---
categories:
  - CSS
tags:
  - css
date: 2018-12-17 23:03:07
title: CSS word-break 속성
updated:
---


글자 줄바꿈을 어떻게 할지 정하는 속성.

1. word-break: normal;

default 값으로 영어의 경우 띄어쓰기 단위로 줄바꿈 되지만 한글은 중간에서 끊긴다.

{% jsfiddle 3y8wzajL html,css,result %}

2. word-break: break-all;

영어, 한글 모두 중간에서 끊긴다.

{% jsfiddle 49y1k0wq html,css,result %}

3. word-break: keep-all;

영어, 한글 모두 띄어쓰기 단위로 끊긴다. 정확히는 영어는 normal로 적용되고 한글이 띄어쓰기 단위로 끊기는 것.

{% jsfiddle uwtk5f8g html,css,result %}

<br>
보다 자세한 설명은 [여기](https://www.w3schools.com/csSref/css3_pr_word-break.asp)에 