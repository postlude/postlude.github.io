---
categories:
  - JAVA
tags:
  - java
date: 2019-08-26 23:03:22
title: Windows에 OpenJDK 설치하기
updated:
---


Oracle JDK의 유료화 이슈(정확히는 기업에서 쓰는 용도가 유료화) 이후로 많은 기업들이 OpenJDK로 변경한 것으로 알고 있습니다.
(사실 회사에서 JAVA를 쓰지 않아서 정확하지는 않습니다만, 예전에 면접볼 때 물어본 기업은 변경해서 큰 이슈없이 잘 쓰고 있다고 했었습니다.)

사실 개인적으로 공부할 때는 상관없지만, 그래도 환경을 맞춰보고자 로컬 환경에 OpenJDK를 설치해보고자 합니다.

## 1. OpenJDK 다운로드

Oracle JDK처럼 공식 사이트에서 받기 위해 구글링을 하던 도중 2개의 사이트를 찾았습니다.

첫 번째는 {% link github 저장소 https://github.com/ojdkbuild/ojdkbuild %}입니다.

{% asset_img 2.JPG %}

두 번째는 {% link 이 사이트 https://jdk.java.net/ %}입니다.
왼쪽의 Archive에서 다운받으실 수 있습니다.

{% asset_img 3.JPG %}

양쪽 다 테스트하기 위해 github 저장소에서는 1.8 버전 msi 파일을 받았고,
jdk 사이트에서는 10.0.2 버전의 tar.gz 파일을 받았습니다.

## 2.1 설치

msi 파일은 간단합니다. 그대로 실행시켜서 설치하시면 됩니다.

{% asset_img 1.JPG %}

tar 파일의 경우에는 원하는 경로에 압축을 풀어 설치합니다.

{% asset_img 4.JPG %}

## 2.2 환경변수 설정

아래와 같은 경로로 접근해 JAVA_HOME 경로를 설정해줍니다.

내 컴퓨터 우클릭 - 속성 - 고급 시스템 설정 - 환경 변수 - 새로 만들기

{% asset_img 5.JPG %}

<br>

{% asset_img 6.JPG %}

<br>

{% asset_img 7.JPG %}

<br>

{% asset_img 8.JPG %}

JAVA_HOME 생성 후 Path를 수정해줍니다.

Path 선택 후 편집 - 새로 만들기

{% asset_img 9.JPG %}

<br>

{% asset_img 10.JPG %}

## 3. 테스트

세팅한 JAVA_HOME을 변경해 가면서 cmd 창에서 아래 명령어를 실행했습니다.

{% codeblock %}
    java -version
    javac -version
{% endcodeblock %}

{% asset_img 11.JPG %}

<br>

{% asset_img 12.JPG %}

8버전과 10버전 모두 정상적으로 세팅되는 것을 확인했습니다.