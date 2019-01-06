---
categories:
  - Git
tags:
  - git
date: 2019-01-06 16:22:21
title: Git Submodule 사용하기
updated:
---

어떤 repo 안에서 다른 repo를 관리하고자 할 때 git submodule을 사용할 수 있다.

예시를 보도록 하자.

## 0. 기본 상태

![](/img/git-submodule-1.JPG)

## 1. submodule로 사용할 디렉토리 추가 및 push

repository를 따로 사용할 수 도 있지만 같은 repository에서 브랜치만 달리해서 submodule을 추가하는 방식을 사용했다.

이를 위해 submodule 디렉토리를 추가하고 파일을 추가했다.

{% codeblock %}
  mkdir submodule
  cd submodule
  touch test.txt
  echo 'test file' >> test.txt
{% endcodeblock %}

{% codeblock %}
  git init
  git checkout -b submodule
  git add .
  git commit -m "add test file"
  git remote add origin https://github.com/postlude/GithubPractice.git
  git push -u origin submodule
{% endcodeblock %}

![](/img/git-submodule-3.JPG)

해당 프로젝트의 동일한 repo의 <code>submodule</code>이라는 브랜치에 해당 내용이 올라간 것을 확인할 수 있다.

## 2. submodule 연결

<code>git submodule add</code> 명령어를 통해 submodule을 추가할 수 있다.

{% codeblock %}
  git submodule add [repo_url] [submodule_directory_name(생략시 repo 이름과 동일하게 생성)]
{% endcodeblock %}

나는 동일한 repository에 브랜치만 다르게 해서 submodule을 다르게 했다.

그러면 아래와 같이 submodule 디렉토리와 <code>.gitmodule</code> 이라는 파일이 생성된 것을 확인할 수 있다.

![](/img/git-submodule-2.JPG)

## 2. submodule init

submodule 디렉토리를 또 하나의 git repository 로 사용하게 해야하므로 <code>git init</code> 과 파일을 하나 추가했다.

