---
categories:
  - K8S
tags:
  - k8s
  - kops
date: 2021-07-07 23:18:19
title: kops 쿠버네티스 클러스터에 cert-manager 설치하기
updated:
---

{% post_link kops-k8s-ingress 지난 포스트 %}에서 ingress를 세팅하고 접속하는 것까지 진행했습니다.
이번에는 클러스터에 cert-manager addon을 설치하는 작업을 진행하도록 하겠습니다.
이 작업을 마치면 ingress를 통해 접속시 https로 접속이 가능해집니다.

## 1. addon 설치

기본적으로 {% link kops 문서 https://kops.sigs.k8s.io/addons/#cert-manager %} 참고하여 진행하겠습니다.

먼저 아래 명령어를 통해 cert-manager 를 활성화합니다.

{% codeblock edit cluster lang:Bash %}
    kops edit cluster
{% endcodeblock %}

{% codeblock set cert-manager addon lang:Bash %}
    ...
    spec:
      certManager:
        enabled: true
    ...
{% endcodeblock %}

이후 update 명령어를 통해 변경될 설정을 적용합니다.

{% codeblock update cluster lang:Bash %}
    kops update cluster --yes
{% endcodeblock %}

적용이 완료되면 **kube-system** 네임스페이스에 cert-manager 관련 pod가 뜬 것을 확인할 수 있습니다.

{% asset_img 1.JPG %}

## 2. issuer 생성

https 통신을 위해서는 인증서가 필요합니다. 이 인증서를 무료로 발급해주는 Let's Encrypt를 사용하도록 하겠습니다.
{% link cert manager 문서 https://cert-manager.io/docs/tutorials/acme/ingress/#step-6-configure-let-s-encrypt-issuer %}에 나와 있는 방법에 따라 진행하도록 하겠습니다.

아래와 같이 staging-issuer.yaml 파일을 작성합니다.

{% codeblock staging-issuer.yaml lang:Bash %}
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-staging
    spec:
      acme:
        server: https://acme-staging-v02.api.letsencrypt.org/directory
        email: [이메일 주소]
        privateKeySecretRef:
          name: letsencrypt-staging
        solvers:
        - http01:
            ingress:
              class:  nginx
{% endcodeblock %}

문서와 다른 점은 `kind: Issuer`가 아닌 `kind: ClusterIssuer` 라는 것입니다.
Issuer는 네임스페이스에 포함된 리소스라 네임스페이스가 다른 ingress에서는 해당 Issuer를 사용하지 못합니다.
따라서 ClusterIssuer로 만들어 모든 네임스페이스에서 사용이 가능하도록 하겠습니다.

아래 명령어로 생성합니다.

{% codeblock create clusterissuer lang:Bash %}
    kubectl apply -f ./staging-issuer.yaml
{% endcodeblock %}

생성 후 `kubectl describe clusterissuers letsencrypt-staging` 명령어를 통해 확인했을때 맨아래에 다음과 같이 Status: True라고 나오면 정상적으로 생성된 것입니다.

{% asset_img 2.JPG %}

## 3. ingress 생성

이전에 test 네임스페이스에 만든 ingress는 삭제하도록 하겠습니다.

{% codeblock delete ingress lang:Bash %}
    kubectl delete ingress test-ingress -n test
{% endcodeblock %}

그리고 아래와 같은 ingress를 생성하도록 하겠습니다. {% link 이 문서 https://cert-manager.io/docs/tutorials/acme/ingress/#step-7-deploy-a-tls-ingress-resource %}를 참고했습니다.

{% codeblock cert-test-ingress.yaml lang:Bash %}
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: cert-test-ingress
      annotations:
        kubernetes.io/ingress.class: "nginx"
        cert-manager.io/cluster-issuer: "letsencrypt-staging"
    spec:
      tls:
      - hosts:
        - [ELB에 연결된 HOST 이름]
        secretName: letsencrypt-staging
      rules:
      - host: [ELB에 연결된 HOST 이름]
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: [test 네임스페이스의 서비스 이름]
                port:
                  number: 80
{% endcodeblock %}

저는 cluster issuer로 만들었기 때문에 annotation이 issuer가 아닌 cluster-issuer로 작성했습니다.
아래 명령어로 생성하겠습니다.

{% codeblock create ingress lang:Bash %}
    kubectl apply -f ./cert-test-ingress.yaml -n test
{% endcodeblock %}

아래 명령어로 확인했을때 마지막 부분에 Status: True라고 나오면 정상적으로 생성된 것입니다.
(몇 초 가량 시간이 걸릴 수 있습니다.)

{% codeblock describe ingress lang:Bash %}
    kubectl describe certificate letsencrypt-staging -n test
{% endcodeblock %}

만약 위 yaml 파일에서 path에 `/`가 아닌 다른 경로를 주었다면 annotation에 `ingress.kubernetes.io/rewrite-target: /`을 추가하면 됩니다.

{% codeblock cert-test-ingress.yaml lang:Bash %}
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: cert-test-ingress
      annotations:
        kubernetes.io/ingress.class: "nginx"
        cert-manager.io/cluster-issuer: "letsencrypt-staging"
        ingress.kubernetes.io/rewrite-target: /
    spec:
      tls:
      - hosts:
        - [ELB에 연결된 HOST 이름]
        secretName: letsencrypt-staging
      rules:
      - host: [ELB에 연결된 HOST 이름]
        http:
          paths:
          - path: /test
            pathType: Prefix
            backend:
              service:
                name: [test 네임스페이스의 서비스 이름]
                port:
                  number: 80
{% endcodeblock %}

ELB에 연결한 도메인에 접속하면 아래와 같이 정상적으로 접속되는 것을 알 수 있습니다.

{% asset_img 3.jpg %}

## 4. prod issuer 적용

위의 과정까지 거치면 정상적으로 접속은 됩니다. 하지만 여전히 인증서가 유효하지 않는 것으로 나옵니다.
이유는 Let's Encrypt 문서에서 확인할 수 있었습니다.

{% blockquote Let's Encrypt https://letsencrypt.org/ko/docs/glossary/#def-staging def-staging %}
    Let’s Encrypt는 속도 제한에 영향을 주지 않고 인증서 요청을 테스트할 수 있는 스테이징 API를 제공합니다.
    스테이징 환경에서 생성된 인증서는 공개적으로 신뢰되지 않습니다.
{% endblockquote %}

따라서 운영 환경에서 쓸 수 있는 issuer를 생성하면 됩니다.
과정의 위의 staging issuer를 만들고 적용하는 과정과 동일합니다.

아래와 같이 prod-issuer.yaml 파일을 생성합니다.

{% codeblock prod-issuer.yaml lang:Bash %}
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        server: https://acme-v02.api.letsencrypt.org/directory
        email: [이메일]
        privateKeySecretRef:
          name: letsencrypt-prod
        solvers:
        - http01:
            ingress:
              class:  nginx
{% endcodeblock %}

{% codeblock create prod issuer lang:Bash %}
    kubectl apply -f ./prod-issuer.yaml
{% endcodeblock %}

아래 명령어를 통해 정상적으로 생성되었는지 체크합니다.

{% codeblock describe prod issuer lang:Bash %}
    kubectl describe clusterissuer letsencrypt-prod
{% endcodeblock %}

test 네임스페이스의 ingress를 수정합니다.

{% codeblock edit ingress lang:Bash %}
    kubectl edit ingress cert-test-ingress -n test
{% endcodeblock %}

{% codeblock cert-test-ingress lang:Bash %}
    ...
    metadata:
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-prod
    ...
    secretName: letsencrypt-prod
    ...
{% endcodeblock %}

이렇게 하면 동일하게 접속했을때 staging 일때와 다르게 https 인증서 경고가 뜨지 않는 것을 확인할 수 있습니다.

{% asset_img 4.jpg %}

이렇게 모든 설정을 마쳤습니다. 쿠버네티스에 배포되는 앱이 추가되면 ingress rule을 추가해 적용하면 됩니다.

## ※ 참고 문서

- {% link ingress annotaion 관련 https://cert-manager.io/docs/usage/ingress/ %}