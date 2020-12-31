---
categories:
  - Docker
tags:
  - docker
date: 2020-12-31 17:47:33
title: docker build시 특정 레이어 이후로만 캐시 비활성화 하기
updated:
---

Dockerfile을 작성해 빌드를 할 때 캐시와 관련된 내용을 정리해보도록 하겠습니다.

## 1. 일반적인 상황

예시로 아래와 같은 Dockerfile을 만들었습니다.

{% codeblock Dockerfile lang:Bash %}
    FROM nginx:latest

    COPY ./test1.txt /

    RUN apt-get update \
        && apt-get install -y vim

    COPY ./test2.txt /

    CMD ["nginx", "-g", "daemon off;"]
{% endcodeblock %}

이 Dockerfile을 아래 명령어로 첫 빌드를 하게 되면 여러 메시지와 함께 빌드가 됩니다.

{% codeblock lang:Bash %}
    docker build -t ngins:test .
{% endcodeblock %}

{% codeblock lang:Bash %}
    Step 1/5 : FROM nginx:latest
    ---> ae2feff98a0c
    Step 2/5 : COPY ./test1.txt /
    ---> 4a3488e1f867
    Step 3/5 : RUN apt-get update     && apt-get install -y vim
    ---> Running in 4c3e3e6b8ce2
    Get:1 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]

    ...

    Step 4/5 : COPY ./test2.txt /
    ---> 3fc5a63670e8
    Step 5/5 : CMD ["nginx", "-g", "daemon off;"]
    ---> Running in 6d4383aa7c6f
    Removing intermediate container 6d4383aa7c6f
    ---> 4f7c8d7872fc
    Successfully built 4f7c8d7872fc
    Successfully tagged nginx:test
{% endcodeblock %}

만약 Dockerfile이 변경되지 않은 상태로 다시 빌드하게 되면 아래와 같이 캐시를 사용하는 것으로 빌드 메시지가 나오게 됩니다.

{% codeblock lang:Bash %}
    Sending build context to Docker daemon  9.216kB
    Step 1/5 : FROM nginx:latest
    ---> ae2feff98a0c
    Step 2/5 : COPY ./test1.txt /
    ---> Using cache
    ---> 4a3488e1f867
    Step 3/5 : RUN apt-get update     && apt-get install -y vim
    ---> Using cache
    ---> ea3bc604e29b
    Step 4/5 : COPY ./test2.txt /
    ---> Using cache
    ---> 3fc5a63670e8
    Step 5/5 : CMD ["nginx", "-g", "daemon off;"]
    ---> Using cache
    ---> 4f7c8d7872fc
    Successfully built 4f7c8d7872fc
    Successfully tagged nginx:test
{% endcodeblock %}

## 2. 캐시 비활성화

위 상황에서 캐시를 사용하지 않으려면 어떻게 해야할까요?
docker build 명령어에 `--no-cache` 옵션을 주면 됩니다.

{% codeblock lang:Bash %}
    docker build --no-cache -t ngins:test .
{% endcodeblock %}

이렇게 하면 처음 빌드를 할 때와 동일하게 모든 레이어를 다시 빌드하게 됩니다.

## 3. 특정 레이어 이후로만 캐시 비활성화

이런 상황을 가정해보겠습니다.

예를 들어 어떤 레이어는 빌드가 오래 걸리지만 변경될 일이 없어서 캐시를 사용해야하고 싶을 수도 있습니다.
(실제로 제가 회사 작업 중에 맞닥뜨린 상황이었습니다.)

`--no-cache` 옵션은 모든 레이어의 캐시를 사용하지 않는 옵션이기 때문에 이 상황에서는 사용할수가 없습니다.

명령어 혹은 옵션으로 해결 방법이 존재하는 것은 아니지만, 우회하여 해결할 수 있는 방법은 있습니다.

바로 ARG 를 통해 argument를 전달하는 것입니다.

{% codeblock Dockerfile lang:Bash  %}
    FROM nginx:latest

    COPY ./test1.txt /

    ARG DISABLE_CACHE

    RUN apt-get update \
        && apt-get install -y vim

    COPY ./test2.txt /

    CMD ["nginx", "-g", "daemon off;"]
{% endcodeblock %}

{% codeblock lang:Bash %}
    CUR_TIME=$(date +%s)
    docker build --build-arg DISABLE_CACHE=$CUR_TIME -t nginx:test .
{% endcodeblock %}

위와 같이 DISABLE_CACHE argument 값을 준 후 빌드를 하면 ARG로 이전의 레이어는 캐시를 사용하고,
이후는 캐시를 사용하지 않게 되어 RUN 으로 실행한 명령어가 실행되게 됩니다.

{% codeblock lang:Bash %}
    Sending build context to Docker daemon  9.216kB
    Step 1/6 : FROM nginx:latest
    ---> ae2feff98a0c
    Step 2/6 : COPY ./test1.txt /
    ---> Using cache
    ---> 4a3488e1f867
    Step 3/6 : ARG DISABLE_CACHE
    ---> Using cache
    ---> b6ab6ed6f1e4
    Step 4/6 : RUN apt-get update     && apt-get install -y vim
    ---> Running in e62eeb8aa185
    Get:1 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]

    ...

    Step 5/6 : COPY ./test2.txt /
    ---> b5019df04f18
    Step 6/6 : CMD ["nginx", "-g", "daemon off;"]
    ---> Running in 946f777160a4
    Removing intermediate container 946f777160a4
    ---> 42787d4961b9
    Successfully built 42787d4961b9
    Successfully tagged nginx:test
{% endcodeblock %}

단, 여기도 몇 가지 제약이 있습니다.

- 캐시 비활성화를 위해 넘겨주는 ARG 는 위의 **시간 값처럼 항상 변하는 값**이어야 합니다.
  고정된 값을 줄 경우 처음에만 캐시를 사용하지 않고, 두 번째 이후부터는 다시 캐시를 사용합니다.
- 이 방법은 `RUN` 명령어 이후만 캐시 비활성화가 가능합니다.
  즉, COPY 에는 적용되지 않기 때문에 ARG 선언부가 RUN 밑으로 오게 되면 'COPY ./test2.txt' 부분은 계속 캐시를 사용하게 됩니다.
  만약 COPY에도 적용하고 싶다면 ARG - RUN - COPY 순으로 Dockerfile을 작성하면 RUN 이후의 COPY까지도 캐시를 사용하지 않게 됩니다.

## 4. 참고

- {% link Selectively disable caching for specific RUN commands in Dockerfile http://dev.im-bot.com/docker-select-caching/ %}