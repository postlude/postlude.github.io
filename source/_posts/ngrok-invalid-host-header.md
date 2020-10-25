---
categories:
  - 기타
tags:
  - tool
date: 2020-10-25 22:50:01
title: ngrok 사용시 Invalid Host header 에러
updated:
---


최근 개인 프로젝트를 진행하던 중 ngrok을 사용할 일이 있었습니다.({% post_link ngrok ngrok 포스트 %})

저는 Vue를 이용해서 프로젝트를 진행중이었고, localhost:8080 으로 띄운 서버를 ngrok을 통해 확인하고 싶었습니다.

{% asset_img 1.JPG %}

이 상황에서 ngrok을 띄웠습니다.

{% codeblock %}
    ngrok.exe http 8080
{% endcodeblock %}

ngrok에 뜬 주소로 접속해보니 아래와 같은 화면이 나왔습니다.

{% asset_img 2.JPG %}

다행히도 검색을 통해 손쉽게 해결 방법을 찾았습니다.

{% blockquote stakoverflow https://stackoverflow.com/questions/45425721/invalid-host-header-when-ngrok-tries-to-connect-to-react-dev-server 링크 %}
    ngrok http 8080 -host-header="localhost:8080"
{% endblockquote %}

위와 같은 명령어로 ngrok을 실행후 접속해보았습니다.

{% asset_img 3.JPG %}

정상적으로 접속되는 것을 확인할 수 있습니다.