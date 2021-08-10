---
categories:
  - K8S
tags:
  - k8s
  - kops
date: 2021-07-05 18:17:59
title: kops로 구축한 k8s 클러스터에 ingress 설정하기
updated:
---

{% post_link kops-k8s-deploy-nginx 지난 포스트 %}에서는 nginx pod를 띄우고 NodePort 타입의 서비스를 통해 접속했습니다.
NodePort로 접속하는 방법은 worker node의 특정 포트로 직접 접속하는 것인데
이번에는 ingress를 이용해 nginx에 접속하는 것으로 변경해보도록 하겠습니다.

ingress는 간단히 설명하자면 kubernetes의 모든 요청을 설정한 룰에 따라 한 곳에서 처리하도록 하는 것입니다.
자세한 설명은 {% link 쿠버네티스 문서 https://kubernetes.io/docs/concepts/services-networking/ingress/ %}를 참고하시면 됩니다.

## 1. ingress addon 설치

{% link kops github https://github.com/kubernetes/kops/tree/97869057129b30ea284c3ed1bdf1db36e752701d/addons/ingress-nginx %}을 보면 ingress-nginx를 설치할 수 있는 addon이 있는 것을 알 수 있습니다.
저는 AWS를 사용중이므로 github에 나와 있는 것과 동일하게 아래 명령어를 사용해 설치해보도록 하겠습니다.

{% codeblock install ingress-nginx addon lang:Bash %}
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/ingress-nginx/v1.6.0.yaml
{% endcodeblock %}

그러면 아래와 같이 Warning이 나오면서 뭔가 제대로 되지 않습니다.

{% asset_img 1.JPG %}

이걸 해결하기 위해서는 해당 yaml파일을 조금 수정해야 합니다.
일단 아래 명령어를 통해 전부 삭제하도록 하겠습니다.

{% codeblock delete ingress-nginx addon lang:Bash %}
    kubectl delete -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/ingress-nginx/v1.6.0.yaml
{% endcodeblock %}

아래 명령어로 yaml 파일을 다운받습니다.

{% codeblock download ingress yaml lang:Bash %}
    curl -Lo ingress-nginx-v1.6.0.yaml https://raw.githubusercontent.com/kubernetes/kops/master/addons/ingress-nginx/v1.6.0.yaml
{% endcodeblock %}

우선, yaml파일의 `v1beta1`로 되어 있는 부분을 전부 `v1`으로 변경합니다. 또한 저는 ingress-nginx replicas를 1로 줄이도록 하겠습니다.

{% codeblock modify yaml lang:yaml %}
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: ingress-nginx
      namespace: kube-ingress
      labels:
        k8s-app: nginx-ingress-controller
        k8s-addon: ingress-nginx.addons.k8s.io
    spec:
      replicas: 1 # 3에서 1로 변경
{% endcodeblock %}

그 후 수정한 yaml 파일로 다시 ingress를 생성합니다.

{% codeblock create ingress-nginx addon lang:Bash %}
    kubectl apply -f ./ingress-nginx-v1.6.0.yaml
{% endcodeblock %}

그럼 아래와 같이 정상적으로 생성된 것을 볼 수 있습니다.

{% asset_img 2.JPG %}

**그리고 중요한 것이 이 ingress를 생성한 순간 AWS의 ELB가 생성되게 됩니다.**

AWS 콘솔의 EC2에서 로드밸런서에서 확인할 수 있으며 ELB는 과금 요소이기 때문에 사용량이 많으면 비용을 소모하게 됩니다.
(물론 개인이 사용할 경우 그렇게 큰 비용이 청구되지는 않는 것 같습니다. 저의 경우 ELB 과금을 포함해도 한 달에 100달러 정도 청구되었습니다.)

## 2. ingress 생성

위 과정으로 통해 ingress controller가 생성되었으니 이제 ingress를 생성해보도록 하겠습니다.
{% post_link kops-k8s-deploy-nginx 이전 포스트 %}에서 만든 test 네임스페이스에 ingress를 생성하고 NodePort가 아닌 ingress를 통해서 접속하도록 세팅하겠습니다.

우선 기존에 NodePort 타입으로 생성되어 있던 서비스를 `ClusterIP` 타입으로 변경하겠습니다.

{% codeblock edit service lang:Bash %}
    kubectl edit svc nginx-deployment -n test
{% endcodeblock %}

{% codeblock edit service lang:Bash %}
    ...
    spec:
      ...
      type: ClusterIP
{% endcodeblock %}

그리고 {% link kubernetes 문서 https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource %}에서 기본적인 ingress yaml 파일을 가져와 아래와 같이 수정했습니다.

{% codeblock test-ingress.yaml lang:yaml %}
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: test-ingress
      annotations:
        ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /test
            pathType: Prefix
            backend:
              service:
                name: nginx-deployment
                port:
                  number: 80
{% endcodeblock %}

{% blockquote %}
    쿠버네티스 문서에 있는 yaml에는 annotaions에 `nginx.ingress.kubernetes.io/rewrite-target: /`으로 설정되어 있습니다.
    해당 annotation은 ingress의 path로 들어왔을때 /(루트)로 들어온 것으로 처리해주는 것입니다. 그런데 이유는 알 수 없으나 이렇게 설정하면 정상적으로 동작하지 않아서 아래와 같이 변경했습니다.
    `ingress.kubernetes.io/rewrite-target: /`
{% endblockquote %}

그리고 아래 명령어를 통해 test 네임스페이스에 ingress를 생성합니다.

{% codeblock create ingress lang:Bash %}
    kubectl apply -f ./test-ingress.yaml -n test
{% endcodeblock %}

아래 명령어를 통해 확인하면

{% codeblock get ingress lang:Bash %}
    kubectl get ingress -n test
{% endcodeblock %}

아래와 같이 접속할 수 있는 url을 확인할 수 있습니다.

{% asset_img 3.jpg %}

test-ingress.yaml에서 path를 /test로 설정했으므로 위 URL/test 로 접속하면 정상적으로 접속되는 것을 확인할 수 있습니다.

{% asset_img 4.jpg %}

ingress로 접속이 되니 이전에 worker node ec2에 NodePort 접속을 위해 포트 오픈한 인바운드 규칙은 제거합니다.

## 3. 도메인에 ELB 연결하기

모든 설정을 완료하긴 했는데 뭔가 찜찜합니다. 접속하는 URL이 너무 지저분합니다.
그래서 이 URL을 처음 k8s를 구축할 때 구매했던 도메인의 서브 도메인에 등록하도록 하겠습니다.

[Route53 - 호스팅 영역 - 구입한 도메인]으로 접속합니다.
레코드 생성을 클릭하고 아래와 같이 설정합니다.

{% asset_img 5.jpg %}

- 레코드 이름을 입력
- 별칭 선택
- [Application/Classic Load Balancer에 대한 별칭] 선택
- 리전 선택
- elb 선택

이렇게 설정하면 [설정한 주소/test]로 접속했을 때 동일하게 nginx welcome 페이지를 볼 수 있습니다.