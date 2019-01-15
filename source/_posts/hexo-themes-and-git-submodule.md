---
categories:
  - Hexo
tags:
  - hexo
  - git
date: 2019-01-01 23:22:43
title: Hexo 테마 적용과 git submodule
updated:
---

hexo로 블로그를 작성하면서 테마를 적용하기 위해선 다음과 같은 식의 명령어로 테마를 적용하게 된다.

{% codeblock %}
  git clone https://github.com/ppoffice/hexo-theme-hueman.git themes/hueman
{% endcodeblock %}

themes 디렉토리 하위에 hueman 이라는 디렉토리가 생기면서 해당 저장소의 내용을 clone 받게 된다.

즉, 아래와 같이 기본적인 설정과 포스트를 작성하는 전체 git repo 안에 다른 repo가 들어간 모양새가 된다.

![](/img/hexo-themes-and-git-submodule-1.JPG)

상위 git repo만 github에 올려 관리를 하던 도중 평소 작업을 하던 곳이 아닌 다른 pc에서 작업을 하기 위해 이 블로그 repo를 clone 받았다.

그런데 이게 왠걸 themes/hueman 디렉토리에 있는 내용은 전혀 받아지지 않았다.

![](/img/hexo-themes-and-git-submodule-2.JPG)

> 이 상태에선 <code>npm i</code> 후에 <code>hexo server</code>를 실행해도 **WARN  No layout: index.html** 과 같은 메시지가 나오면서 아무 페이지도 뜨지 않습니다.

혹시나 싶어 github에 들어가보니 다음과 같이 아예 선택이 불가능한 상태로 되어 있었다.

![](/img/hexo-themes-and-git-submodule-3.JPG)

그제야 커밋을 하고 푸쉬를 할 때 뭔가 평소와 달랐던 것이 생각났다.

어떻게 해야 이 문제를 해결할 수 있을까?
<br>

## 1. 원인

간단하다. 하나의 git 저장소 안에 또다른 git 저장소가 들어있을 경우 한꺼번에 관리되지 않는다.

## 2. 상태 및 목표

현재 나는 블로그에 관련된 코드는 하나의 repo에서 브랜치만 달리해서 관리하고 있다.

(처음 시작할 때 repo를 따로 두는 것을 고민하긴 했는데 아무래도 한 곳에 있는게 관리하기 편할 것 같아 그렇게 결정했다.)

{% codeblock %}
  master 브랜치 : 실제 배포되는 코드가 올라가는 브랜치
  포스트용 브랜치 : 포스트를 작성하는 브랜치
  설정 변경용 브랜치 : 여러 설정들을 테스트하거나 변경하기 전에 적용해보기 위한 브랜치
{% endcodeblock %}

이 상태에서 <code>themes/hueman</code> 과 같이 테마용 브랜치를 추가해 관리하는 것을 목표로 삼았다.

## 2. 해결 방법 (1)

어차피 cli로 themes/hueman 로 들어가면 **일반적인 git repo**와 동일하게 인식하므로 따로 따로 관리해주면 된다.

#### 2.1. 로컬에서 변경 사항을 커밋 후 remote repo를 설정해준다.

{% codeblock %}
  git remote remove origin
  git remote add origin [github repo 주소]
{% endcodeblock %}

#### 2.2. 원격 저장소에 push

{% codeblock %}
  git push -u origin themes/hueman
{% endcodeblock %}

이렇게 하면 동일 repo의 themes/hueman 브랜치로 <code>themes/hueman</code> 에 있는 내용이 올라가게 된다.

#### 2.3. clone

새로운 곳에서 clone을 받을 때는 부모 repo와 자식 repo를 각각 clone 받아야 한다.

나같은 경우에는 자식 repo가 주소는 같고 브랜치만 다르므로 다음과 같이 옵션을 주어서 clone 받아야 한다.

{% codeblock %}
  git clone -b [브랜치 명] [github repo 주소] [디렉토리 명]
{% endcodeblock %}

여기서 한가지 더 필요한 것이 맨 마지막에 써준 디렉토리 명이다.

그냥 clone을 받게 되면 github repo 명으로 clone을 받게 된다.

![](/img/hexo-themes-and-git-submodule-4.JPG)

내가 원하는 것은 hueman 디렉토리 하위에 바로 clone 받는 것이므로 다음과 같이 명령어를 실행했다.

{% codeblock %}
  git clone -b themes/hueman https://github.com/postlude/postlude.github.io.git themes/hueman
{% endcodeblock %}

이렇게 하면 다음과 같이 깔끔하게 clone 받을 수 있게 된다.

![](/img/hexo-themes-and-git-submodule-5.JPG)

## 3. 해결 방법 (2) - git submodule 이용

해결 방법을 찾던 도중 git에 submodule이라는 것이 있다는 것을 알게 되었다.

> {% post_link git-submodule %}

submodule을 사용하면 부모 repo와 자식 repo가 연결된다는 장점이 있다.

또한 github에서 부모 repo에서 바로 자식 repo의 내용을 확인할 수 있다.

단점은 자식 repo 내용을 수정하고자 할 때 단순 clone과 달리 <code>git checkout</code> 으로 브랜치를 만들어줘야 한다.

## 4. 결론

블로그 repo에서 테마 repo를 사용하는 것이므로 의미적으로는 submodule을 사용하는게 맞을 것 같다.

근데 생각보다 불편하다.. 명령어를 사용하는 것도 오히려 더 많이 써야 하고.

다른 경우라면 모르겠지만 블로그 코드를 관리하는데 있어서는 조금 더 고민을 해봐야 할 것 같다.