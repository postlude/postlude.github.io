---
categories:
  - Docker
tags:
  - docker
date: 2020-12-26 22:58:48
title: Jenkins를 docker 컨테이너로 구축하기(Docker in Docker)
updated:
---

## 0. 계기

올해 초에 회사 jenkins 서버를 docker로 재구축한 적이 있었습니다. 그 때의 경험을 블로그에 반드시 남겨야겠다고 생각했었는데, 이제서야 글을 쓰게 되었네요.

일단 제가 겪은 상황은 다음과 같습니다.

- docker 로 jenkins 서버를 구축합니다.
- 단, jenkins 공식 이미지가 아니라 ubuntu를 기본 이미지로 해 직접 설치하는 식으로 Dockerfile을 작성합니다.
- jenkins에서는 docker build를 실행하도록 설정되어야 합니다.

이 과정에서 제가 겪은 내용들을 정리해보고자 합니다.
**(잘못된 내용이 있다면 댓글로 지적해주시면 감사하겠습니다.)**

## 1. 과정

jenkins 이미지를 만드는 것은 그다지 어렵지 않았습니다.

그런데 docker build를 실행하는 jenkins job을 만들 때 문득 이런 생각이 들었습니다.

{% blockquote %}
    jenkins에서 docker build를 하려면 docker 명령어를 써야하는데 그러면 docker 안에 docker를 설치해야 하나?
{% endblockquote %}

## 2. Docker in Docker

구글링을 하던 도중 {% link 아주 좋은 글 https://aidanbae.github.io/code/docker/dinddood/ %}을 발견했습니다.

간단하게 정리하면 다음과 같습니다.

{% blockquote %}
    docker 안에 docker daemon을 설치하는 것은 가능하지만, 권장하지 않는다.
    목적은 컨테이너 내부에서 docker 명령어를 사용하는 것이므로 호스트의 도커 명령어를 사용할 수 있도록 세팅한다.
{% endblockquote %}

## 3. 세팅

처음에는 구글링해서 나오는 여러 글들처럼 docker run 명령어 사용시에 아래와 같은 세팅을 추가했습니다.

{% codeblock lang:bash  %}
    docker run \
    -v /var/run/docker.sock:/var/run/docker.sock \
    ...
{% endcodeblock %}

젠킨스에는 docker 관련 플러그인들이 존재하기 때문에 젠킨스 컨테이너에 위의 세팅 후 플러그인 설치 및 세팅을 거치면

플러그인에서 제공하는 docker 관련 기능들을 사용할 수 있게 됩니다.

다만, 플러그인에서 모든 도커 관련 명령어에 대한 기능을 제공해주지는 않아서 제 경우에는 직접 도커 명령어를 젠킨스 컨테이너 내부에서 사용이 가능해야 했습니다.

그런데 이렇게 설정을 해도 컨테이너 내부에서 docker 명령어 사용시 `command not found` 라는 메시지만 나왔습니다.

이리 저리 찾아보다가 결국 컨테이너 내부에서도 **docker daemon**이 아닌 **docker cli**는 설치해야 한다는 것을 알게 되었습니다.
({% link docker: not found after mounting /var/run/docker.sock https://stackoverflow.com/questions/55315528/docker-not-found-after-mounting-var-run-docker-sock %})

그에 따라 Dockerfile에 docker를 설치하는 내용을 그대로 작성하되 **docker-ce-cli** 만 설치하도록 작성했습니다.

{% codeblock lang:bash  %}
    apt-get install -y docker-ce-cli
{% endcodeblock %}

※ 참고 : {% link docker-ce / docker-ce-cli / containerd.io 의 차이점 https://www.reddit.com/r/docker/comments/dsr6y2/containerdio_vs_dockercecli_vs_dockerce_what_are/ %}

이렇게 한 결과 컨테이너 내부에서도 도커 명령어를 사용할 수 있게 되었습니다.

정확히는 호스트의 도커 데몬으로 명령을 내리는 것이기 때문에 컨테이너 내부에서 `docker images` 명령어를 치면 호스트의 이미지 목록이 나오게 됩니다.

만약 위의 docker run -v 옵션 없이 docker-ce-cli만 설치 후 도커 명령어를 사용하면 도커 데몬에 연결할 수 없다는 메시지가 나오게 됩니다.
(Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?)

## 4. 권한 설정

위의 세팅까지 적용하면 '젠킨스 컨테이너에서 플러그인으로 도커 빌드하기' + '컨테이너 내부에서 도커 명령어 사용하기' 까지는 가능합니다.

젠킨스 job에서 직접 도커 명령어를 사용하려고 하면 아마 권한 문제로 실행이 되지 않을 겁니다.

그 이유는 컨테이너 내부/외부의 /var/run/docker.sock 을 보면 알 수 있습니다.

호스트의 /var/run/docker.sock 은 다음과 같습니다.

{% asset_img 2.JPG %}

반면에 컨테이너 내부는 다음과 같습니다.

{% asset_img 1.JPG %}

컨테이너 내부에는 docker 라는 그룹이 없습니다. 따라서 호스트의 그룹 아이디(994)가 그대로 나오게 됩니다.

또한, jenkins 유저는 root도 아니고 994 라는 그룹에 속해있지도 않기 때문에 docker 명령어를 사용할 수 없는 것입니다.

컨테이너 내부에 994 아이디로 docker 라는 그룹을 만들고 jenkins 유저를 docker 그룹에도 속하게 함으로써 젠킨스 job에서도 도커 명령어를 사용할 수 있게 되었습니다.

## 5. 결론

당시에 이 문제를 해결할 때는 꽤 시간을 잡아먹었는데 그래도 얻은 것도 많은 좋은 경험이었습니다.

참고한 링크들도 남겨드리니 한 번쯤 보시는 것도 좋을 것 같습니다.

- {% link DinD(docker in docker)와 DooD(docker out of docker) https://aidanbae.github.io/code/docker/dinddood/ %}
- {% link Using Docker-in-Docker for your CI or testing environment? Think twice. http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/ %}
- {% link Docker-in-Docker https://github.com/jpetazzo/dind %}
- {% link Docker in Docker? https://itnext.io/docker-in-docker-521958d34efd %}