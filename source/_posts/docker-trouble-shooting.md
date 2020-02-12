---
categories:
  - null
tags:
  - null
date: 2020-02-12 23:44:57
title:
updated:
---


최근 회사에서 사용하던 도커 버전을 업그레이드했습니다.

오늘은 그 과정에서 겪었던 Trouble Shooting을 적어보려고 합니다.

## 1. 컨테이너 구동 시 아래와 같은 에러가 발생할 경우

{% blockquote %}
    docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "process_linux.go:430: container init caused \"write /proc/self/attr/keycreate: permission denied\"": unknown.
{% endblockquote %}

구글링을 통해 알아본 결과 container-selinux package의 버전이 무언가 맞지 않아 생긴 문제로 보였습니다.

따라서 해당 패키지를 update(혹은 downgrade)를 해서 해결했습니다.

{% codeblock %}
    yum list container-selinux
 
    yum update container-selinux
    yum downgrade container-selinux
{% endcodeblock %}

검색한 다른 해결 방법으론 selinux를 끄는 방법도 있었지만, 왠지 내키지 않아 시도해보지는 않았습니다.
참고1 : {% link 특정 버전으로 설치 https://stackoverflow.com/questions/56870478/cannot-start-docker-container-in-docker-ce-on-oracle-linux %}
참고2 : {% link containerd를 1.2.5로 downgrade https://gitmemory.com/issue/containerd/containerd/3862/560205832 %} (해당 방법으로는 직접해보지는 않아서 확실하진 않습니다.)

## 2. docker images 에는 이미지 목록이 나오는데 docker rmi 명령어로 이미지를 지울 경우 'No such image' 메시지가 나올 경우

버전 업을 하면서 어딘가 꼬여서 이렇게 나오는 것으로 보입니다.

도커를 내리고 `/var/lib/docker` 디렉토리를 삭제 후 다시 실행하면 됩니다.
(`/var/lib/docker` 디렉토리는 도커 실행시 다시 생성됩니다.)

{% codeblock %}
    systemctl stop docker
 
    # /var/lib/docker 디렉토리는 docker 엔진 재시작시 새로 생성됨
    rm -rf /var/lib/docker
    
    systemctl start docker
{% endcodeblock %}

참고 : {% link stackoverflow https://stackoverflow.com/questions/46381888/docker-images-shows-image-docker-rmi-says-no-such-image-or-reference-doe/46386670 %}