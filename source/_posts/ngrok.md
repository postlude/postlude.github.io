---
categories:
  - 기타
tags:
  - tool
date: 2019-08-20 23:54:37
title: ngrok 사용방법
updated:
---


이번 포스팅은 개발시 유용한 툴에 대해 소개하고자 합니다.

로컬 환경에서 개발을 할 때 이런 경험이 있으실 겁니다.

- 개발한 화면을 실제 모바일 기기에서 확인해보고 싶을 때
- 로컬 환경에 접속할 수 없는 외부에서 개발 내용을 확인하고 싶을 때

이를 위해서는 실제로 서버를 띄워서 확인해야하는데, 시간도 걸리고 여러모로 번거로운 면이 있습니다.
이런 경우에 사용하는 툴이 바로 **ngrok** 입니다.

## 1. 다운로드

{% link 여기 https://ngrok.com %}서 다운 받으실 수 있습니다.

{% asset_img 1.JPG %}

{% asset_img 2.JPG %}

저는 윈도우에서 사용을 할 것이므로 윈도우용 ngrok을 다운 받았습니다.
다운을 받고 압축을 풀면 정말 심플하게 **ngrok.exe** 파일 하나가 있습니다.

## 2. 테스트용 코드 작성

테스트를 위해서 아주 간단한 node 코드를 작성했습니다.

{% codeblock ngrok-test lang:JavaScript  %}
	const express = require('express');
	const app = express();
	const morgan = require('morgan');
	const port = 3000;

	app.use(morgan('dev'));

	app.get('/', (req, res) => {
		res.send('test app');
	});

	app.listen(port, () => {
		console.log(`========== [Node Test Server Start(port: ${port})] ==========`);
	});
{% endcodeblock %}

이 코드를 실행시킨 후 로컬에서 접속하면 다음과 같은 화면을 보게 됩니다.

{% asset_img 3.JPG %}

## 3. ngrok 테스트

이제 ngrok을 사용해보겠습니다. 
윈도우 cmd창을 열어서 ngrok.exe가 설치된 경로로 이동합니다.
그리고 아래와 같은 명령어를 실행합니다.
(마지막 숫자는 포트 번호입니다. 저는 샘플 코드를 3000 포트로 동작시켰기 때문에 3000으로 적었습니다.)

{% codeblock %}
	ngrok.exe http 3000
{% endcodeblock %}

{% asset_img 4.JPG %}

위와 같은 화면이 나오면서 ngrok이 동작합니다. 위에 표시된 주소로 접근하게 되면

{% asset_img 5.JPG %}

위와 같이 동일한 화면을 볼 수 있게 됩니다.
이 주소는 localhost로 접속이 되지 않는 곳에서도 접속이 가능합니다.
(단, ngrok이 떠있는 동안에만 접속할 수 있습니다.)

또한, ngrok을 정지시킨 후 다시 구동시키면 주소는 변경됩니다.
