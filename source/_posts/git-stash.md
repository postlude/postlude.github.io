---
categories:
  - Git
tags:
  - git
date: 2019-01-20 18:49:54
title: git stash 사용하기
updated:
---

혼자서 작업을 할 때는 사실 stash 기능을 많이 쓸 일은 없었다. 간혹 파일 수정하던 내용을 전부 날릴 때 정도?

그런데 회사에 들어와 작업을 하다보니 내가 작업한 내용을 로컬에 저장하고 다른 작업을 하다가 다시 불러와 작업하는

말 그대로 stash 기능을 제대로 사용할 일이 종종 생겼다.

그래서 이번 기회에 제대로 정리해보고자 한다.

# 0. 기본 상태

{% asset_img git-stash-1.JPG %}

이 상태에서 stash를 여러 파일 상태에 적용해보기 위해 다음과 같은 상태로 만들었다.

{% asset_img git-stash-2.JPG %}

* 기존 파일 수정(file1)
* 새 파일(file2, file3) 추가
* 추가한 파일 중 하나만 add 해 stage 상태로 두었다.

# 1. git stash

기본적으로는 <code>git stash</code> 명령어만으로도 작성중이던 내용을 저장할 수 있다.

{% asset_img git-stash-3.JPG %}

그러면 위와 같이 `git stash list`로 저장된 내용을 확인할 수 있다.

그런데 `git status`로 확인한 내용은 내 생각과는 약간 달랐다. file3은 저장되지 않고 그대로였다.

일단 나중에 다시 보기로 하고 stash에 저장한 내용을 다시 불러와보자.

`git stash apply [stash 번호]`로 stash에 있는 내용을 적용할 수 있다.

{% asset_img git-stash-4.JPG %}

그러면 처음 상태로 파일을 되돌릴 수 있다.

# 2. stash untracked file

stash 명령어를 실행했을 때 왜 file3은 stash에 저장되지 않았을까?

이유는 다음과 같았다.

> 기본적으로 `git stash` 명령어는 git이 tracking 하고 있는 파일에 한해 적용된다.

즉, file3은 새로 추가되어 untracked file 이라 stash 로 저장되지 않은 것이다.

file2는 **git add 명령어를 통해 staging area에 있었기 때문에** stash 명령어로 저장이 될 수 있었다.

그럼 이렇게 새로 만든 파일을 stash로 저장하는 방법이 없을까?

당연히(?) 있다.

{% codeblock %}
  git stash -u
{% endcodeblock %}

{% asset_img git-stash-5.JPG %}

untracked file인 file3까지 저장되었다.

동일하게 `git stash apply`명령어로 복원 가능하다.

# 3. git stash save

위 상태에서 `git stash list`로 저장된 목록을 보면 다음과 같다.

{% asset_img git-stash-6.JPG %}

맨 처음에 stash로 저장한 내용을 삭제하지 않았기 때문에 저장한 목록이 2개다. 그런데 0번과 1번중 어떤게 먼저 저장한 것이고 어떤 게 나중에 저장한 것일까?

정답은 0번이 나중에 저장한 내용이다.

즉, **가장 최근에 stash로 저장한 내용이 0번이 된다.**

그런데 보기 불편하다. 커밋처럼 메시지를 주고 싶다.

역시 가능하다.

{% codeblock %}
  git stash save 'stash message'
{% endcodeblock %}

일단 `apply`로 원복시킨 다음에 `save`했다.

{% asset_img git-stash-7.JPG %}

`save` 명령어도 마찬가지로 `-u` 옵션을 주어야 untracked file도 저장된다.

# 4. stash push / pop

`save / apply` 와 유사하게 `push / pop` 명령어도 존재한다.

차이점이 있다면 `pop` 을 하면 stash list에서 해당 내용이 사라진다.

또한 `push`를 할 때 `save`처럼 메시지를 주고 싶다면 `-m`옵션을 사용해야 한다.
(untracked file을 저장하기 위해서는 동일하게 `-u`옵션을 사용한다.)

역시 기본 상태로 돌려놓고 명령어를 실행했다.

{% asset_img git-stash-8.JPG %}

# 5. save vs. push

stash를 공부하던 도중 다음과 같은 자료를 발견했다.

{% blockquote 7.3 Git 도구 - Stashing과 Cleaning https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Stashing%EA%B3%BC-Cleaning %}
`git stash push` 로의 이동
2017년 10월 말 Git 메일링 리스트에는 엄청난 논의가 있었습니다. 논의는 `git stash save` 명령을 은퇴시키고 `git stash push`로 대체하는 내용에 대한 것이었습니다.
`git stash push` 명령의 경우 pathspec 으로 선택하여 Stash하는 옵션이 추가되었는데 `git stash save` 명령이 지원하지 못하는 것이었습니다.

`git stash save` 명령이 곧바로 삭제되는 것은 아니기에 아직 이 명령을 쓰는 것에 대해 걱정할 필요는 없지만 `git stash push` 명령으로 대체하는 것에 대해 생각해볼 필요가 있습니다.
{% endblockquote %}

그럼 대체 pathspec이 뭐길래 그럴까?

왠지 path라는 단어에서 느낌이 오기 시작한다. help 문서에도 다음과 같이 나와있다.

{% blockquote %}
  When pathspec is given to git stash push, the new stash entry records the modified states only for the files that match the pathspec. The index entries and working tree files are then rolled back to the state in HEAD only for these files, too, leaving files that do not match the pathspec intact.
{% endblockquote %}

그렇다. stash로 저장할 때 특정 파일만 저장하는 것이 가능하다는 말이다.

마찬가지로 기본 상태에서 명령어를 실행했다.

{% asset_img git-stash-9.JPG %}
{% asset_img git-stash-10.JPG %}

# 6. 요약

{% blockquote %}
  * `git stash save` : stash 저장. 개별 파일들을 따로 저장은 불가
  * `git stash apply` : stash 복원. 복원시 저장된 내용이 삭제되지 않는다.
  * `git stash push` : stash 저장. 개별 파일 저장 가능.
  * `git stash pop` : stash 복원. 복원시 저장된 내용이 삭제된다.
  * `git stash list` : stash로 저장된 내용 확인.
  * `git stash clear` : stash에 저장된 내용 전부 삭제
  * `git stash drop` : 개별 stash 저장 내용을 삭제
{% endblockquote %}

# ※ 번외 테스트

이 글을 쓰던 도중 문득 궁금해졌다.

stash가 기본적으로 git이 tracking하는 파일만 적용하기 때문에 add 한 파일까지 적용되는 건데,

**만약 add 한 파일을 다시 원복시킨 후에는 stash 명령어로 저장이 될까?**

{% asset_img git-stash-11.JPG %}
{% asset_img git-stash-12.JPG %}

혹시나 했지만 역시나 되지 않는다.