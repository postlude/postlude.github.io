---
categories:
  - 기타
tags:
  - linux
date: 2019-02-19 11:06:53
title: VirtualBox에서 CentOS 설치하기 (1)
updated:
---


(벼르고 벼르던) VirtualBox 세팅에 관한 포스팅을 시작하겠습니다.
우선 시작하기에 앞서 포스팅의 목적은 제가 성공한 방법을 정리하고 전달하고자 함입니다.
혹 더 좋은 세팅이나 방법을 알고 계시면 전달해주시면 감사하겠습니다.

{% blockquote %}
    이 글에서 작성한 내용은 대체로 경험에 의존한 것입니다. 구글링을 해서 찾은 내용을 바탕으로 하긴 했습니다만 근거가 명확하지 않을 수 있습니다.
{% endblockquote %}

# 0. 목적

Windows 환경에서 리눅스를 사용하기 위함입니다. 사실 VirtualBox를 이용한 방법 말고도 AWS나 WSL(Windows Subsystem for Linux)을 이용하는 방법도 있습니다.
(그리고 사실 더 간단하긴 합니다..)
이전에 {% post_link windows-ubuntu 다른 포스트 %}에서 WSL을 이용하는 방법도 포스팅했으니 참고하시면 될 것 같습니다.

다만, WSL의 경우 docker를 사용하기는 어려운 것 같았습니다. 찾아보니 아직 완전한 리눅스 시스템이 아니라서 그런 것 같습니다.
(구글링 했을 때 방법은 있었습니다만 호스트(windows)에서 docker를 똑같이 돌리는 방법으로 봤습니다. 이럴바에야 그냥 window용 도커를 사용하는 것이 나을 것 같았습니다. 그런데 window용 도커는 또 호스트와 마운트가 잘 안되는 것 같더군요. 결국은 로컬에서 도커 테스트시에는 virtualbox를 써야되는 것 같습니다.)

# 1. 다운로드

우선 VirtualBox를 다운받습니다. ({% link 링크 https://www.virtualbox.org/wiki/Downloads %})
VirtualBox는 기본 설치만 할 경우 화면 크기 조정이나 클립보드 공유 등의 기능이 제대로 되지 않습니다. 따라서 VirtualBox뿐만 아니라 Extension Pack도 함께 설치하셔야 합니다.

{% asset_img 2.JPG %}

Linux는 CentOS 7을 사용하겠습니다. ({% link 링크 http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso %})
미러 중에서 아무거나 받아주시면 됩니다.

{% asset_img 1.JPG %}

# 2. VirtualBox 설치

VirtualBox를 먼저 설치한 후 Extension Pack을 설치해주시면 됩니다.
VirtualBox 설치시 라이센스 동의는 스크롤을 맨 밑으로 내리면 '동의합니다'가 활성화됩니다.

{% asset_img 3.JPG %}

# 3. CentOS 설치

## 3.1. 호스트 키 조합 설정

본격적으로 CentOS를 설치하기 전에 사전 세팅을 하나만 하도록 하겠습니다.
가상머신과 호스트의 마우스 인식을 전환하는 버튼이 기본적으로는 [Right Control] 로 되어 있습니다. 그런데 막상 설치해보면 잘 동작하지 않습니다. 이 키 세팅을 변경하도록 하겠습니다.

[파일 - 환경설정 - 입력 - 가상 머신] 으로 들어가면 다음과 같이 '호스트 키 조합' 이 있습니다. 이 조합을 (좌측)Ctrl + Alt 로 변경하겠습니다.

{% asset_img 16.JPG %}

사실 최종적인 세팅을 마치면 가상머신과 호스트 사이를 마우스가 왔다갔다 할 때 아무런 키를 누르지 않아도 됩니다. 하지만 이 설정을 변경하지 않으면 설치시 상당히 불편해지기 때문에 변경하도록 하겠습니다.

## 3.2. 가상 머신 설정

이제 CentOS를 설치하도록 하겠습니다. 새로 만들기를 선택해주시면 됩니다.
(캡쳐를 좀 잘못해서 이미 머신이 생성되어 있는 것으로 나왔는데 기본적으로는 아무것도 없습니다.)

{% asset_img 5.JPG %}

가상 머신의 이름을 입력합니다. 나중에 수정가능하니 크게 신경쓰지 않으셔도 됩니다.
버전 같은 경우도 크게 상관은 없지만 기왕이면 redhat으로 맞춰주도록 하겠습니다.

{% asset_img 6.JPG %}

메모리를 설정합니다. 중요한 것은 **2G 이하로 설정할 경우 가상 머신이 굉장히 느릴 수 있습니다.**
반면에 너무 메모리를 너무 많이 설정하면 호스트(Windows)가 느려질 수 있습니다. 제 노트북의 메모리는 8G이므로 4G로 설정하겠습니다.

{% asset_img 7.JPG %}

디스크 사이즈를 설정합니다. 20G 정도로 할당해두니 크게 문제없이 사용가능했습니다.

{% asset_img 8.JPG %}
{% asset_img 9.JPG %}
{% asset_img 10.JPG %}
{% asset_img 11.JPG %}

부팅 디스크를 선택합니다. 사전에 다운받은 CentOS 이미지를 선택하면됩니다.

{% asset_img 12.JPG %}

호스트 키를 변경하기 전에 캡쳐를 해서 Right Control 로 표시 되었는데, 앞서 변경하셨다면 Ctrl + Alt로 표시가 될 겁니다.
[잡기]를 선택하시면 됩니다.

{% asset_img 13.JPG %}

## 3.3. CentOS 설치

이제 CentOS 를 설치하겠습니다. 키보드 위 방향키를 한 번 눌러서 Install CentOS 7을 선택하시면됩니다.
(기본적으로 2번째 Test가 선택되어 있기 때문에 바로 엔터키를 누르시면 안됩니다.)

{% asset_img 14.JPG %}

언어 선택입니다. 저는 기본적으로 영어를 선택했습니다.
{% blockquote %}
    이 시점부터는 마우스가 보이는 게 정상입니다. 저도 보이지 않아서 당황했었는데요, 간단하게 설명드리면 [가상 머신 설정 - 시스템 - 마더보드 - 포인팅 장치]에서 변경해주시면 됩니다.
    이에 대한 부분은 다른 포스트에서 여러 팁과 함께 다룰 예정입니다.
{% endblockquote %}
아직까지는 마우스가 호스트로 자연스럽게 빠져나가지 않을 겁니다. 아까 설정한 호스트 키 조합(Ctrl + Alt)으로 Windows로 마우스를 빠져나갈 수 있습니다.

> 여기서부터는 설치 중인 가상 머신을 강제 종료할 경우 이어서 설치가 되지 않습니다. 처음부터 다시 설치하셔야 합니다.

{% asset_img 15.JPG %}

설정에 관련된 세팅들입니다. 여기서 DATE & TIME, SOFTWARE SELECTION, NETWORK & HOST NAME 정도만 세팅하겠습니다.

{% asset_img 17.JPG %}

먼저 시간 관련 세팅입니다. Asia / Seoul 로 세팅하겠습니다.

{% asset_img 18.JPG %}

SOFTWARE SELECTION 입니다. **Server with GUI를 선택**하셔야 합니다. Minimal로 설치했을 때는 이후 '게스트 확장 이미지'를 설치하는 과정에서 제대로 되지 않았습니다.
(혹시 Minimal로도 가능한 방법을 아시는 분은 알려주시면 감사하겠습니다.)

{% asset_img 19.JPG %}

네트워크 세팅은 화면 우측에 현재 네트워크의 토글 버튼이 있을텐데 활성화 해주시면 됩니다. 활성화할 경우 가상 머신이 부팅될 때 자동으로 네트워크에 연결됩니다.
사실 설치 이후에 설정도 가능한 부분이라 크게 중요하진 않습니다.

{% asset_img 27.JPG %}

INSTALLATION DESTINATION은 파티션 분할을 하는 내용입니다. 저는 크게 관여하지 않을 것이라 이미 선택된 Autometic partitioning으로 두겠습니다.
Done 을 눌러주시면됩니다.

{% asset_img 29.JPG %}
{% asset_img 30.JPG %}


설정을 마쳤으니 Begin Installation 을 선택하시면 됩니다.

{% asset_img 20.JPG %}

설치가 되는 동안 root 계정에 대한 패스워드와 사용자 계정을 생성할 수 있습니다.
사용자 계정을 생성하지 않아도 부팅시에 자동으로 하나의 계정을 생성하게 되므로 여기서 생성하도록 하겠습니다.
**Make this user administrator**를 선택하게 되면 해당 유저가 sudo 권한을 가지게 됩니다.

{% asset_img 28.JPG %}

설치가 완료되면 Reboot을 선택합니다.

{% asset_img 21.JPG %}

재부팅되면 OS를 선택하는 화면이 나옵니다. 위쪽을 선택하시면 됩니다.
아래쪽은 rescue 모드라고 해서 윈도우의 안전모드와 비슷한 모드라고 합니다.

{% asset_img 22.JPG %}

라이센스에 대한 내용도 체크해주시면 됩니다.

{% asset_img 23.JPG %}
{% asset_img 24.JPG %}

드디어 CentOS 설치를 마쳤습니다. 이렇게 사용하셔도 문제는 없습니다만 화면 크기 조정이나 여러 부분에서 불편한 점이 있기 때문에 다른 세팅들을 좀 더 할 예정입니다.

{% post_link virtualbox-centos-install-2 다음 포스트 %}에 이어집니다.

{% asset_img 26.JPG %}