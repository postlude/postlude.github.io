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

# 0. 기본 상태

{% asset_img git-submodule-1.JPG %}

# 1. Submodule로 사용할 디렉토리 추가 및 push

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

{% asset_img git-submodule-3.JPG %}

해당 프로젝트의 동일한 repo의 <code>submodule</code>이라는 브랜치에 해당 내용이 올라간 것을 확인할 수 있다.

# 2. Submodule 연결

<code>git submodule add</code> 명령어를 통해 submodule을 추가할 수 있다.

{% codeblock %}
  git submodule add [repo_url] [submodule_directory_name(생략시 repo 이름과 동일하게 생성)]
{% endcodeblock %}

기본적으로 <code>git submodule add</code> 명령어를 이용하면 해당 repo를 clone 받으면서 submodule을 추가하게 된다.
하지만 나는 submodule로 사용할 repo를 로컬에 먼저 작성하고 push를 한 다음에 submodule만 추가하는 방법으로 진행했기 때문에 아래와 같은 명령어를 사용했다.

{% codeblock %}
  git submodule add -b submodule https://github.com/postlude/GithubPractice.git submodule
{% endcodeblock %}

`clone`과 유사하게 `-b` 옵션을 이용해 브랜치를 지정해주고, 디렉토리명 또한 지정해주었다.
그러면 <code>.gitmodules</code> 이라는 파일이 생성된 것을 확인할 수 있다.
.gitmodule의 내용을 보면 다음과 같다.

{% asset_img git-submodule-4.JPG %}

# 3. Project commit

이제 부모 프로젝트에서 해당 내용들을 반영해보자.

{% asset_img git-submodule-5.JPG %}

이 내용을 github에서 확인해보면 다음과 같다.

{% asset_img git-submodule-6.JPG %}
{% asset_img git-submodule-7.JPG %}

단순히 repo 자체를 넣었을 때와는 다르게 자식 repo의 내용을 github에서 확인할 수 있다.

# 4. Submodule 수정

submodule 디렉토리 하위에 test2.txt라는 파일을 만들었다.
부모 repo에서 `git add .` 후에 다시 status를 봐도 색깔이 변하지 않았다.

{% asset_img git-submodule-2.JPG %}

즉, **자식 repo에서 commit한 내용만 부모에서 반영할 수 있다.**

{% asset_img git-submodule-8.JPG %}
{% asset_img git-submodule-9.JPG %}

정상적으로 커밋되었다.
이제 이 내용을 push하고 확인해보자.

{% asset_img git-submodule-10.JPG %}
{% asset_img git-submodule-11.JPG %}

커밋 내용을 보면 정상적으로 올라간 것 같은데 subdmodule 내용이 보이지 않는다.
당연하다. **submodule을 push하지 않았기 때문이다.**
submodule 디렉토리로 이동해 push하고 확인하면 정상적으로 내용을 확인할 수 있다.

{% asset_img git-submodule-12.JPG %}

# 5. Submodule이 포함된 repo clone 하기

submodule이 포함된 디렉토리를 clone 받으면 submodule에는 아무런 내용도 들어있지 않다.

{% asset_img git-submodule-13.JPG %}

명령어를 통해 submodule을 clone 받는다.

{% codeblock %}
  git submodule init
  git submodule update
{% endcodeblock %}

{% asset_img git-submodule-14.JPG %}

> git submodule update 를 하면 위와 같이 clone을 받는다.
<br>

# ※주의 사항※

git submodule update 한 후에 submodule 디렉토리에 들어가면 다음과 같이 되어 있다.

{% asset_img git-submodule-15.JPG %}

정상적인 작업 브랜치로 보이지 않는다. 찾아보니 다음과 같이 설명되어 있다.

{% blockquote git-scm.com https://git-scm.com/book/ko/v1/Git-%EB%8F%84%EA%B5%AC-%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88#%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%ED%95%A0-%EB%95%8C-%EC%A3%BC%EC%9D%98%ED%95%A0-%EC%A0%90%EB%93%A4 서브모듈 사용할 때 주의할 점들 %}
  git submodule update 명령을 실행시키면 특정 브랜치가 아니라 슈퍼프로젝트에 저장된 커밋을 Checkout해 버린다. 그러면 detached HEAD라고 부르는 상태가 된다. detached HEAD는 HEAD가 브랜치나 태그 같은 간접 레퍼런스를 가리키지 않고 커밋을 가리키는 것을 말한다. 데이터를 잃어 버릴 수도 있기 때문에 일반적으로 detached HEAD 상태는 피해야 한다.
{% endblockquote %}

그러므로 아래처럼 브랜치를 만들어 이동해준다.

{% codeblock %}
  git checkout -b submodule
{% endcodeblock %}

<br>
<hr/>
참고 문서 : [git-scm.com](https://git-scm.com/book/ko/v1/Git-%EB%8F%84%EA%B5%AC-%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88)


