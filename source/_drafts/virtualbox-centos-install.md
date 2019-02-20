---
categories:
  - null
tags:
  - null
date: 2019-02-19 11:06:53
title: virtualbox-centos-install
updated:
---

(벼르고 벼르던) VirtualBox 세팅에 관한 포스팅을 시작하겠습니다.
우선 시작하기에 앞서 포스팅의 목적은 제가 성공한 방법을 정리하고 전달하고자 함입니다.
혹 더 좋은 세팅이나 방법을 알고 계시면 전달해주시면 감사하겠습니다.

{% blockquote %}
    이 글에서 작성한 내용은 대체로 경험에 의존한 것입니다. 구글링을 해서 찾은 내용을 바탕으로 하긴 했습니다만 근거가 명확하지 않을 수 있습니다.
{% endblockquote %}

## 0. 목적

Windows 환경에서 리눅스를 사용하기 위함입니다. 사실 VirtualBox를 이용한 방법 말고도 AWS나 WSL(Windows Subsystem for Linux)을 이용하는 방법도 있습니다.
(그리고 사실 더 간단하긴 합니다..)
이전에 {% post_link windows-ubuntu 다른 포스트 %}에서 WSL을 이용하는 방법도 포스팅했으니 참고하시면 될 것 같습니다.

다만, WSL의 경우 docker를 사용하기는 어려운 것 같았습니다. 찾아보니 아직 완전한 리눅스 시스템이 아니라서 그런 것 같습니다.
(구글링 했을 때 방법은 있었습니다만 호스트(windows)에서 docker를 똑같이 돌리는 방법으로 봤습니다. 이럴바에야 그냥 window용 도커를 사용하는 것이 나을 것 같았습니다. 그런데 window용 도커는 또 호스트와 마운트가 잘 안되는 것 같더군요. 결국은 로컬에서 도커 테스트시에는 virtualbox를 써야되는 것 같습니다.)

## 1. 설치


