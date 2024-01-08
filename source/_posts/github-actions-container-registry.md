---
categories:
  - Git
tags:
  - git
  - github
  - docker
date: 2021-07-20 22:05:56
title: GitHub Actions와 GitHub Container Registry를 이용한 도커 빌드
updated:
---

이번 포스트에서는 GitHub Actions와 GitHub Container Registry를 이용해서 CI를 구축해보도록 하겠습니다.

# 1. 선택의 이유

회사에서는 CI를 젠킨스를 이용해 구축해서 사용중입니다. 그래서 개인적으로 사용할 때도 젠킨스를 이용한 방법을 사용하려고 했습니다.
그러던 중 몇 가지 이유로 GitHub Actions와 GitHub Container Registry를 선택하게 되었습니다.

- 젠킨스 세팅 문제
젠킨스에서 docker build가 가능하도록 세팅하는 것은 {% post_link docker-in-docker 다른 포스트 %}에서도 해봤기 때문에 어려운 문제는 아니었습니다.
다만, `docker run` 을 실행할 때 호스트의 docker를 사용 가능하도록 옵션을 주는 부분이 뭔가 깔끔하지 않다고 느껴졌습니다.
그리고 이걸 쿠버네티스 위에 올려서 사용해야하는데 그 부분도 뭔가 찝찝하기도 했고요.
(물론 제가 모르는 젠킨스 컨테이너에서 도커 빌드가 가능하도록 하는 방법이 있을수도 있습니다.)

- docker hub의 repository 제한 및 정책 변경
사실 개인적으로만 쓰는 용도기 때문에 docker hub의 repository가 많이 필요한 것은 아닙니다. private repository도 무료로 쓰는 유저는 1개까지만 만들 수 있지만 이것도 그렇게 불편한 것은 아닙니다.
그렇다고 하더라도 약간은 신경이 쓰이긴 했습니다.
여기에 더해 docker hub가 최근 무료 계정의 이미지에 대해 몇 가지 정책을 변경했다고 합니다.({% link 관련 내용 https://subicura.com/k8s/2021/01/02/docker-hub-pull-limit/ %})

- AWS ECR
ECR 또한 고려했었습니다. 많진 않겠지만 추가 비용이 발생하는 것이 신경이 쓰였습니다.

여기에 더해 다음 이유로 github의 기능들을 선택하게 되었습니다.

1. github actions + github container registry = github에서 모두 처리가 가능
2. 쿠버네티스에 추가적으로 젠킨스와 같은 빌드용 pod를 띄울 필요가 없음
3. 제일 큰 이유는 바로 비용에 관한 것입니다. 저는 이미 github을 pro로 사용중이기 때문에 추가 비용이 없습니다.

# 2. 기본 세팅

우선 빌드에 사용할 아주 간단한 node 서버를 만들어보도록 하겠습니다.

패키지는 아래와 같이 3개만 설치하겠습니다.

{% codeblock npm install lang:Bash %}
    npm i express morgan pm2
{% endcodeblock %}

그리고 아래와 같이 app.js와 ecosystem.config.js 파일을 작성합니다.

{% codeblock app.js lang:JavaScript %}
    const express = require('express');
    const morgan = require('morgan');

    const app = express();
    const port = 3000;

    app.use(express.json());
    app.use(morgan('dev'));

    app.listen(port, async () => {
        console.log('==================== [NODE SAMPLE] ====================');
        console.log(`- PORT : ${port}`);
        console.log('========================================================');
    });
{% endcodeblock %}

{% codeblock ecosystem.config.js lang:JavaScript %}
    module.exports = {
        apps: [{
            name: 'node-sample',
            script: './app.js',
            watch: true,
            ignore_watch: ['.git', 'log', 'node_modules'],
            log_date_format: 'YYYY-MM-DD HH:mm:ss',
            out_file: './log/pm2_out.log',
            error_file: './log/pm2_err.log'
        }]
    };
{% endcodeblock %}

# 3. Personal Access Token 생성

github container regitstry에 이미지를 push 하는 등의 작업을 위해서는 PAT가 필요합니다.

[github setting - Developer settings - Personal access tokens] 로 들어가 Generate new token을 클릭합니다.

{% asset_img 1.JPG %}

아래와 같이 권한을 선택하고 생성합니다.

{% blockquote %}
    - write:packages
    - read:packages
    - delete:packages
{% endblockquote %}

{% asset_img 2.JPG %}

이제 이 토큰을 이용해 github container registry에 push 하거나 pull 할 수 있습니다.
예를 들어 굳이 github actions를 이용하지 않더라도 개인적으로 빌드한 도커 이미지를 `docker push`를 이용해 push 할 수 있다는 이야기입니다.
(단, docker 로그인시 토큰을 비밀번호로 사용하여 로그인해야 합니다.)

아래는 예시 코드입니다.

{% codeblock sample code lang:Bash %}
    docker login https://ghcr.io -u [계정명]
    docker tag node-sample:latest ghcr.io/[계정명]/node-sample:latest
    docker push ghcr.io/[계정명]/node-sample:latest
{% endcodeblock %}

# 4. Dockerfile 작성

이제 샘플 코드의 app.js와 동일한 위치에 Dockerfile을 작성하도록 하겠습니다.

{% codeblock Dockerfile lang:Dockerfile %}
    FROM node:14

    ENV SERVER_HOME /usr/src/node-sample

    RUN mkdir ${SERVER_HOME}

    COPY . ${SERVER_HOME}

    WORKDIR ${SERVER_HOME}

    RUN npm i \
        && npx pm2 install pm2-logrotate@latest \
        && npx pm2 set pm2-logrotate:rotateInterval '0 0 * * *'

    CMD ["npx", "pm2-runtime", "start", "ecosystem.config.js"]
{% endcodeblock %}

# 5. GitHub Actions workflow 작성

workflow에서 container registry에 push 하기 위해서는 위에서 만든 PAT가 필요합니다.
이걸 workflow에 직접 넣는 것이 아니라 repository에서 사용 가능한 secret을 만들어 사용하도록 하겠습니다.
[repository의 Settings - Secrets] 로 들어가 New repository secret을 클릭합니다.

{% asset_img 5.JPG %}

원하는데로 Name을 입력하고 Value에는 PAT 값을 입력합니다. 저는 GHCR_PAT 라는 이름으로 생성했습니다.

이제 github actions workflow를 작성하도록 하겠습니다.
repository의 Actions 탭으로 들어가 New workflow를 클릭합니다.

{% asset_img 3.JPG %}

제일 하단의 Manual workflow 를 선택합니다.

{% asset_img 4.JPG %}

아래와 같이 입력하도록 하겠습니다.

{% codeblock workflow.yml lang:yml %}
    # workflow 이름
    name: Docker Build Test
    
    on:
      push:
        # master branch에 push했을 때 workflow가 돌게 됨
        branches: [master]

    jobs:
      # job 이름. job은 여러 개가 설정되어 돌아갈 수 있음.
      node-sample-build:
        runs-on: ubuntu-latest
        steps:
        - name: Source Code Checkout
          uses: actions/checkout@master
        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v1
        - name: Login to GitHub Container Registry
          uses: docker/login-action@v1
          with:
            registry: ghcr.io
            username: ${{ github.repository_owner }}
            password: ${{ secrets.GHCR_PAT }}
        - name: Build and push
          uses: docker/build-push-action@v2
          with:
            context: .
            file: ./Dockerfile
            push: true
            tags: ghcr.io/${{ github.repository_owner }}/node-sample:master
{% endcodeblock %}

workflow 작성에 관한 자세한 사항은 {% link github 문서 https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions %}를 통해 확인할 수 있습니다.

github에서 workflow를 작성하게 되면 `.github/workflows/` 디렉토리 하위에 작성한 yml 파일이 생성됩니다.
변경 사항이 생겼으므로 `git pull` 후 아무 변경 사항을 추가한 후에 push 하면 workflow가 돌게 됩니다.

workflow가 도는 것은 repository의 Actions 탭에서 확인할 수 있습니다.

{% asset_img 6.JPG %}

정상적으로 push까지 완료되었으므로 Packages 에서 확인할 수 있습니다.

{% asset_img 7.JPG %}

# ※ 참고 문서

- {% link GitHub 문서 https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions %}
- {% link GitHub Container Registry 사용하기 https://blog.outsider.ne.kr/1530 %}
- {% link GitHub Actions에서 GitHub Container Registry에 이미지 푸시하기 https://blog.outsider.ne.kr/1531 %}
- {% link Github Actions를 이용한 Docker Image Build 및 Push https://teichae.tistory.com/entry/Github-Actions%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-Docker-Image-Build-%EB%B0%8F-Push %}