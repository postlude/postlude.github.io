---
categories:
  - Docker
tags:
  - k8s
  - kops
date: 2021-07-03 17:53:54
title: 쿠버네티스 클러스터에 nginx pod 띄우기
updated:
---

{% post_link k8s-by-kops 지난 포스트 %}에서 kops로 kubernetes 클러스터를 구축했었습니다.
이번에는 해당 클러스터에 간단한 nginx 를 띄워보도록 하겠습니다.

# 1. 클러스터 세팅

먼저 클러스터에 몇 가지 세팅을 해주도록 하겠습니다.
(이 부분들은 반드시 필요한 것은 아닙니다.)

## 1.1. 보안 그룹 변경

우선, 저와 동일하게 public 네트워크 형태로 클러스터를 구축하셨다면 쿠버네티스의 모든 master, worker node가 아래와 같이 모든 ip에 대해 ssh 접속이 가능하도록 되어있을겁니다.

{% asset_img 1.jpg %}

이 부분을 저희 집과 bastion 서버에서만 접속이 가능하도록 수정하도록 하겠습니다.

{% asset_img 2.jpg %}

## 1.2. 백업 설정 변경

다음은 백업에 관한 설정입니다.
{% link 문서 https://kops.sigs.k8s.io/operations/etcd_backup_restore_encryption/#taking-backups %}에 따르면 hourly backup은 1주일간 유지되며 daily backup은 1년간 유지된다고 합니다.
백업 내용은 클러스터 구축시 만들었던 S3에 저장됩니다.

저는 개인적으로 클러스터를 구축한 것이고 (아마도) 백업 파일이 많아지면 S3 비용 부담이 생길지도 몰라서 이 기간을 줄이도록 하겠습니다.
{% link 이 문서 https://kops.sigs.k8s.io/cluster_spec/#etcd-backups-retention %}를 참고했습니다.

bastion 서버에서 아래 명령어를 입력합니다.

{% codeblock edit cluster lang:Bash %}
    kops edit cluster
{% endcodeblock %}

그러면 클러스터 설정을 세팅할 수 있도록 나올텐데요, 아래와 같이 변경하도록 하겠습니다.

{% codeblock set backup lang:Bash %}
  etcdClusters:
  - cpuRequest: 200m
    etcdMembers:
    - encryptedVolume: true
      instanceGroup: master-ap-northeast-2a
      name: a
    memoryRequest: 100Mi
    name: main
    # ===== 추가 =====
    manager:
      env:
      - name: ETCD_MANAGER_HOURLY_BACKUPS_RETENTION
        value: 1d
      - name: ETCD_MANAGER_DAILY_BACKUPS_RETENTION
        value: 7d
    # ===== 추가 =====
  - cpuRequest: 100m
    etcdMembers:
    - encryptedVolume: true
      instanceGroup: master-ap-northeast-2a
      name: a
    memoryRequest: 100Mi
    name: events
    # ===== 추가 =====
    manager:
      env:
      - name: ETCD_MANAGER_HOURLY_BACKUPS_RETENTION
        value: 1d
      - name: ETCD_MANAGER_DAILY_BACKUPS_RETENTION
        value: 7d
    # ===== 추가 =====
{% endcodeblock %}

edit 명령어는 설정자체를 저장한 것이고 이것을 실제로 적용하려면 아래 명령어를 실행해야 합니다.

{% codeblock update cluster lang:Bash %}
    kops update cluster --yes
{% endcodeblock %}

# 2. 샘플 nginx pod 생성

이제 테스트 네임스페이스를 만들어서 nginx pod 를 띄워보도로 하겠습니다.

먼저 아래 명령어로 테스트 네임스페이스를 만듭니다.

{% codeblock create namespace lang:Bash %}
    kubectl create ns test
{% endcodeblock %}

그리고 nginx pod를 띄우기 위한 deployment yaml 파일을 생성합니다.
{% link kubernetes 문서 https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment %}에서 참고했습니다.
다만, 저는 예시로 샘플을 만들기 위함이므로 pod 개수는 1개로 생성하도록 하겠습니다.

{% codeblock nginx-deployment.yaml lang:yaml %}
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.14.2
            ports:
            - containerPort: 80
{% endcodeblock %}

명령어를 통해 deployment를 생성합니다.

{% codeblock create deployment lang:Bash %}
    kubectl apply -f ./nginx-deployment.yaml -n test
{% endcodeblock %}

{% asset_img 3.JPG %}

kubectl 명령어를 통해 deployment와 pod가 생성된 것을 확인할 수 있습니다.

{% codeblock lang:Bash %}
    kubectl get deploy -n test
    kubectl get po -n test
{% endcodeblock %}

{% asset_img 4.JPG %}

# 3. 서비스 생성

이렇게 생성한 pod에 외부에서 접속하기 위해서는 서비스가 필요합니다.
아래 명령어를 통해 서비스를 생성합니다. 외부에서 접속하기 위함이므로 NodePort 타입으로 생성하겠습니다.

{% codeblock create service lang:Bash %}
    kubectl expose deployment nginx-deployment -n test --type NodePort
    kubectl get svc -n test
{% endcodeblock %}

저는 worker node가 1개 뿐이고 NodePort 타입으로 생성했으니 worker node ec2 ip에 생성된 NodePort로 접속이 가능합니다. 그런데 막상 접속해보니,

{% asset_img 5.JPG %}

접속이 되지 않았습니다. {% link 구글링 https://stackoverflow.com/questions/45543694/kubernetes-cluster-on-aws-with-kops-nodeport-service-unavailable %}을 통해 이유를 찾을 수 있었는데요, 원인은 간단합니다.
kops로 생성한 kubernetes 클러스터의 ec2 worker node는 NodePort 타입의 서비스에서 사용하는 범위의 포트(30000-32767)를 자동으로 오픈시켜주지 않기 때문입니다.

포트를 열어주기 위해 ec2 콘솔로 접속해 worker node ec2의 보안그룹을 수정하도록 하겠습니다.

{% asset_img 6.JPG %}

이렇게 수정하면 정상적으로 접속되는 것을 확인할 수 있습니다.

{% asset_img 7.JPG %}
