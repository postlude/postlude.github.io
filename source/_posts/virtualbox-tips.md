---
categories:
  - 기타
tags:
  - linux
date: 2019-03-11 15:35:51
title: VirtualBox 사용 시 유용한 몇 가지 팁들
updated:
---

<br>
{% post_link virtualbox-centos-install-2 지난 포스트 %}까지 virtualbox에 centos 설치하는 방법까지 알아보았습니다.
이번 포스트에서는 centos에서 작업할 때 유용한 미세먼지(?) 팁 들을 알려드리고자 합니다.

## 1. 유용한 Virtual Box 기능
#### 1.1. 복제

가장 많이 쓰는 기능 중에 하나입니다.
테스트하기 위해 리눅스용 환경을 만들었다고 가정해보죠. 열심히 테스트를 한 후에 다른 테스트를 하려고 합니다. 이때 지금까지 테스트한 게스트 os를 남겨두고 싶다거나 재사용이 불가능한 상황이라면 처음부터 환경 세팅을 해야하는 끔찍한 일이 발생할수도 있습니다.
이럴 때는 처음 환경을 구성해 둔 다음에 복제를 해서 사용하는 것이 편합니다.

사용 방법은 간단합니다.

복제하고자 하는 os를 우클릭 - 복제를 선택합니다.

{% asset_img 1.jpg %}

Mac 주소 값을 선택합니다. 새로운 주소로 생성하겠습니다.

{% asset_img 2.JPG %}

복제 방식을 선택합니다. 완전한 복제를 선택하겠습니다.

{% asset_img 3.JPG %}

끝입니다! 간단하죠? 조금 시간이 지나면 복제된 OS가 생성됩니다. 저 같은 경우는 위의 캡쳐와 같이 가장 기본이 되는 OS만 설치한 것을 계속 복제해서 사용하고 있습니다.

#### 1.2. 스냅샷

복제와 유사하지만 조금 다른 스냅샷이라는 기능도 있습니다.
이 기능은 하나의 게스트 OS에서 중간 저장과 같이 스냅샷을 찍어두고 불러올 수 있는 기능입니다.

간단한 예시를 보겠습니다.

루트 디렉토리 하위에 test 라는 디렉토리를 만들었습니다.

{% asset_img 4.JPG %}

이 상태에서 스냅샷을 찍겠습니다. 게스트 os 옆의 버튼을 클릭하고 스냅샷을 선택합니다.

{% asset_img 5.JPG %}

찍기를 선택합니다.

{% asset_img 6.JPG %}

어떤 시점의 스냅샷인지를 파악하기 위한 정보를 기입하는 창이 나옵니다. 테스트이니 그대로 진행하겠습니다.

{% asset_img 7.JPG %}

제가 지정한 이름(스냅샷 1)으로 스냅샷이 저장되었습니다.

{% asset_img 8.JPG %}

아까 만들었던 /test 디렉토리 하위에 test_file 이라는 이름의 파일을 하나 생성하겠습니다.

{% asset_img 9.JPG %}

실행 중이던 virtualbox를 종료하고 스냅샷을 만들었던 창을 보면 Restore 라는 버튼이 활성화되어 있습니다. 이걸 선택합니다.

{% asset_img 10.JPG %}

현재 상태 스냅샷은 굳이 만들진 않겠습니다.

{% asset_img 11.JPG %}

복구한 게스트 os를 실행하면 이전에 제가 작업한 화면으로 복구됩니다.

{% asset_img 12.JPG %}

/test 하위에 test_file도 존재하지 않습니다.

{% asset_img 13.JPG %}

## 2. 마우스 커서가 동작하지 않을 때

이번에 virtualbox에서 리눅스를 설치할 때 한 가지 당황했던 점이 있었습니다.
바로 마우스 커서가 보이지 않는 것이었습니다. 그러다보니 설치 시에 모든 선택을 탭으로 이동하여(...) 작업했었는데요, 구글링을 통해 해결 방법을 찾았습니다.

> 게스트 OS를 실행시키지 않은 상황에서만 설정 변경이 가능합니다.

방법은 간단합니다. 게스트 OS 설정으로 들어가 **[시스템 - 마더보드 - 포인팅 장치]**에서 설정을 변경해주시면 됩니다.

{% asset_img 14.JPG %}

## 3. 터미널 설정
#### 3.1. 화면 꺼짐 설정

virtualbox를 이용해 작업을 하다보면 일정 시간이 지났을 때 화면이 잠기게 됩니다. 이 경우 비밀번호를 다시 입력해야되서 상당히 귀찮습니다.
그래서 개인적으로는 화면을 항상 켜짐 상태로 설정해둡니다.

{% asset_img 15.JPG %}

#### 3.2. 터미널 백그라운드 색상 설정

기본적으로 터미널은 흰 바탕에 검은색 글씨입니다. 흰 바탕색을 오래 보다 보면 눈이 너무 아프기 때문에 배경색을 변경하는 편입니다.
터미널에서 **[Preferences - Colors]**에 들어가서 시스템 테마 사용을 체크 해제하고 원하는 색상이나 테마를 선택하시면 됩니다.

{% asset_img 16.JPG %}

<br>