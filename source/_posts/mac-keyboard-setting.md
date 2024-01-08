---
categories:
  - 기타
tags:
  - etc
date: 2024-01-08 22:13:07
title: Mac에서 키보드 세팅 변경하기 2
updated:
---

{% post_link mac-key-change 예전 글 %}에서 맥 키보드 세팅을 변경하는 걸 정리했었다.

이전 글은 맥에 윈도우용 외부 키보드를 연결해서 쓸 때를 기준으로 쓴 것이라 이번에는 맥 기본 키보드 세팅 변경 글을 써보려고 한다.

{% blockquote %}
	이전 글에서 세팅한 내용은 그대로 유지
{% endblockquote %}

# 1. 설정 변경

[설정 - 키보드 - 키보드 단축키]

{% asset_img 1.png %}

[보조키 - Apple 내장 키보드 / 트랙패드] 에서 Control과 Command를 변경한다.

{% asset_img 2.png %}

# 2. Karabiner-Elements 설정

외부 키보드를 쓸 때와 내장 키보드를 쓸 때를 구분하기 위해 profile을 만든다.

{% asset_img 3.png %}

이제 이 프로파일을 만든 채로 세팅을 하면 프로파일 변경만으로 그에 맞는 세팅이 적용된다.

[Simple Modifications]에서 키세팅을 아래와 같이 적용한다.

{% asset_img 4.png %}

[Complex Modifications]에서 추가 설정 변경을 적용한다.(이건 외부 키보드 세팅과 같다.)

{% asset_img 5.png %}

# 3. AltTab

맥에서도 윈도우에서 쓰던 alt + tab을 동일하게 쓸 수 있는 앱이 있다.

{% asset_img 6.png %}

위와 같이 세팅했기 때문에 누르는 키 위치도 거의 동일하다.

{% link 여기 https://alt-tab-macos.netlify.app/ %}서 다운받을 수 있다.
