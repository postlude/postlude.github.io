---
categories:
  - 기타
tags:
  - nodejs
date: 2021-01-08 18:59:09
title: pm2 watch가 동작하지 않을 때
updated:
---

제가 개인 프로젝트를 위해 신규 Node.js 프로젝트를 만들고 pm2를 이용해 환경을 세팅했는데
pm2 watch가 제대로 동작하지 않은 적이 있습니다. 이에 대한 포스팅을 해보고자 합니다.

## 1. 세팅

일단 기본적으로 아주 간단한 node 서버를 만들었습니다.

{% codeblock package.json lang:JavaScript %}
    {
        "name": "test1",
        "version": "1.0.0",
        "description": "",
        "main": "app.js",
        "scripts": {
        },
        "author": "",
        "license": "ISC",
        "dependencies": {
            "express": "^4.17.1"
        }
    }
{% endcodeblock %}

{% codeblock app.js lang:JavaScript %}
    const express = require('express');

    const app = express();
    const port = 3000;

    app.listen(port, async () => {
        console.log('test1 start');
    });
{% endcodeblock %}

pm2는 글로벌로 설치했으며, 버전은 4.5.0입니다.
ecosystem 파일을 생성합니다.

{% codeblock lang:Bash %}
    pm2 ecosystem
{% endcodeblock %}

ecosystem 파일에는 watch 설정만 했습니다.

{% codeblock ecosystem.config.js lang:JavaScript %}
    module.exports = {
        apps: [{
            name: 'test1',
            script: 'app.js',
            watch: true
        }],
    };
{% endcodeblock %}

## 2. 테스트

위 상태에서 ecosystem 파일을 이용해 서버를 구동합니다.

{% codeblock lang:Bash %}
    pm2 start ecosystem.config.js
{% endcodeblock %}

로그를 보면 정상적으로 구동된 것을 확인할 수 있습니다.

{% codeblock lang:Bash %}
    pm2 log
{% endcodeblock %}

이 상태에서 코드를 수정하면 자동적으로 pm2가 재시작하면서 수정된 코드가 반영됩니다.

{% codeblock app.js lang:JavaScript %}
    app.listen(port, async () => {
        console.log('test1 start');
        console.log('modify code');
    });
{% endcodeblock %}

{% codeblock lang:Bash %}
    PM2      | Change detected on path app.js for app test1 - restarting
    PM2      | Stopping app:test1 id:0
    PM2      | App [test1:0] exited with code [1] via signal [SIGINT]
    PM2      | pid=412 msg=process killed
    PM2      | App [test1:0] starting in -fork mode-
    PM2      | App [test1:0] online
    0|test1  | test1 start
    0|test1  | modify code
{% endcodeblock %}

그런데 pm2 stop을 하고 다시 start하면 watch가 동작하지 않아서 코드를 수정해도 restart가 일어나지 않습니다.

{% codeblock lang:Bash %}
    pm2 stop ecosystem.config.js // 정지 후
    pm2 start ecosystem.config.js // 다시 시작
{% endcodeblock %}

## 3. 해결

좀 더 세련된 방법이 있을지는 모르겠으나 어쨌든 제가 찾아서 해결한 방법은 다음과 같습니다.

{% blockquote %}
    `pm2 stop`후 `pm2 delete`를 통해 프로세스를 아예 삭제 후 start하면 watch가 정상적으로 동작합니다.
{% endblockquote %}

{% codeblock lang:Bash %}
    pm2 stop ecosystem.config.js && pm2 delete ecosystem.config.js // 정지 및 삭제
    pm2 start ecosystem.config.js // 다시 시작
{% endcodeblock %}

## ※ 참고

- https://pm2.keymetrics.io/docs/usage/application-declaration/
- https://github.com/Unitech/pm2/issues/3578