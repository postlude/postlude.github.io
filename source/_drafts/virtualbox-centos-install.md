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

#### 1.1 다운로드

우선 VirtualBox를 다운받습니다. ({% link 링크 https://www.virtualbox.org/wiki/Downloads %})
VirtualBox는 기본 설치만 할 경우 화면 크기 조정이나 클립보드 공유 등의 기능이 제대로 되지 않습니다. 따라서 VirtualBox뿐만 아니라 Extension Pack도 함께 설치하셔야 합니다.

{% asset_img 2.JPG %}

Linux는 CentOS 7을 사용하겠습니다. ({% link 링크 http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso %})
미러 중에서 아무거나 받아주시면 됩니다.

{% asset_img 1.JPG %}

#### 1.2 VirtualBox 설치

VirtualBox 먼저 설치한 후 Extension Pack을 설치해주시면 됩니다.
VirtualBox 설치시 라이센스 동의는 스크롤을 맨 밑으로 내리면 '동의합니다'가 활성화됩니다.

{% asset_img 3.JPG %}

#### 1.3 CentOS 설치

본격적으로 CentOS를 설치하기 전에 사전 세팅을 하나만 하도록 하겠습니다.
가상머신과 호스트의 마우스 인식을 전환하는 버튼이 기본적으로는 Right Control 로 되어 있습니다. 그런데 막상 설치해보면 잘 동작하지 않습니다. 이 키 세팅을 변경하도록 하겠습니다.

[파일 - 환경설정 - 입력 - 가상 머신] 으로 들어가면 다음과 같이 '호스트 키 조합' 이 있습니다. 이 조합을 (좌측)Ctrl + Alt 로 변경하겠습니다.

{% asset_img 16.JPG %}

사실 최종적인 세팅을 마치면 가상머신과 호스트 사이를 마우스가 왔다갔다 할 때 아무런 키를 누르지 않아도 됩니다. 하지만 이 설정을 변경하지 않으면 설치시 상당히 불편해지기 때문에 변경하도록 하겠습니다.

이제 CentOS를 설치하도록 하겠습니다. 새로 만들기를 선택해주시면 됩니다.
(캡쳐를 좀 잘못해서 이미 머신이 생성되어 있는 것으로 나왔는데 기본적으로는 아무것도 없습니다.)

{% asset_img 5.JPG %}

가상 머신의 이름을 입력합니다. 나중에 수정가능하니 크게 신경쓰지 않으셔도 됩니다.
버전 같은 경우도 크게 상관은 없지만 기왕이면 redhat으로 맞춰주도록 하겠습니다.

{% asset_img 6.JPG %}

메모리를 설정합니다. 중요한 것은 `2G 이하로 설정할 경우 가상 머신이 굉장히 느릴 수 있습니다.`
반면에 너무 메모리를 너무 많이 설정하면 호스트(Windows)가 느려질 수 있습니다. 제 노트북의 메모리는 8G이므로 4G로 설정하겠습니다.

{% asset_img 7.JPG %}

디스크 사이즈를 설정합니다. 20G 정도로 할당해두니 크게 문제없이 사용가능했습니다.

{% asset_img 8.JPG %}
{% asset_img 9.JPG %}
{% asset_img 10.JPG %}
{% asset_img 11.JPG %}

부팅 디스크를 선택합니다. 사전에 다운받은 CentOS 이미지를 선택하면됩니다.

{% asset_img 12.JPG %}




