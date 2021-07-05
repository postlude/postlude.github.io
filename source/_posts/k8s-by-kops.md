---
categories:
  - K8S
tags:
  - k8s
  - kops
date: 2021-06-22 23:22:42
title: kops를 이용해 쿠버네티스 클러스터 구축하기
updated:
---

## 0. 안내

이 글은 부정확한 정보를 담고 있을 수 있습니다.
글 내용 중 잘못된 정보나 더 좋은 방법을 알고 계신 분들은 댓글로 알려주시면 감사하겠습니다.

## 1. 배경

kubernetes를 공부하고 싶은 생각은 전부터 있었습니다. 다만, 항상 방법이 문제였습니다.
일단 {% link minikube https://minikube.sigs.k8s.io/docs/start/ %}가 있었기 때문에 기본적인 부분에 대한 공부는 가능했습니다.
minikube는 기초적인 기능을 공부하는데는 좋으나
구축 자체를 연습하거나 worker node를 다룰수는 없기 때문에 해당 부분을 연습하지 못한다는 것은 단점으로 느껴졌습니다.

그 다음으로 선택한 것은 AWS에서 제공하는 EKS였습니다.
일단 무작정 EKS를 구축해보고 여러 설정을 변경해가며 연습해보기 위해 문서를 보며 무작정 구축을 시도했습니다.
시행착오 끝에 EKS를 구축하는데는 성공했었습니다. 다만 문제가 있었는데요, 바로..

{% asset_img 1.JPG %}

**요금이었습니다..**

어떤 형태든 kubernetes를 구축하면 비용이 나갈 것은 각오하고 있었고, 제가 허용 가능한 범위는 달에 10만원 ~ 최대 15만원 정도 였습니다.
EKS를 구축할 때도 대략적인 금액 계산은 하고 비용을 최소화하기 위한 조치(worker node를 1개로 하는 등)를 하고 구축을 하긴 했었는데요,
예상치 못한 부분이 위 이미지에서도 볼 수 있는 NAT Gateway 금액이었습니다.
EKS를 구축하면 VPC를 설정하게 되고 이 과정에서 자동으로 NAT Gateway 쓰게 되면서 과금이 되는 것 같습니다.
(vpc 세팅 중 public과 private를 모두 사용하는 것으로 사용하면 일단 저처럼 NAT Gateway를 사용하게 됩니다. public만 사용하게 설정한다면 어떻게 될지는 모르겠네요.)

일단 요금이 제 예상을 훨씬 웃돌게 되면서 일단 바로 EKS는 삭제했습니다.(...)
그리고 다른 방법을 찾기 시작했습니다. 그러던 중 찾은게 `kubeadm`과 `kops`입니다.
그 중에 저는 kops로 k8s를 구축하기로 했습니다. 이유는 2가지였습니다.

첫 번째로 공식 문서에 'AWS가 공식적으로 지원된다'고 나와 있으며,
두 번째로 요금을 계산해보니 월에 10만원 초반대로 이용이 가능할 것 같았기 때문입니다.

## 2. 기본 세팅

일단 저는 단순히 kubernetes를 '구축하는 것'을 목표로 두었습니다.
여러 세팅의 상세한 의미나 방법은 구축 후에 공부하고, 일단 제가 원하는 정도까지만 구축하는 것으로 방향을 잡았습니다.

기본적으로 {% link 공식 문서 https://kops.sigs.k8s.io/getting_started/install/ %}를 보고 진행했습니다.

### 2.1. IAM 계정 생성

제일 처음 작업은 AWS 작업시 필요한 IAM 계정 생성입니다.

{% link 문서 https://kops.sigs.k8s.io/getting_started/aws/#setup-iam-user %}에 따르면 아래와 같은 권한이 필요하다고 합니다.

{% blockquote %}
    AmazonEC2FullAccess
    AmazonRoute53FullAccess
    AmazonS3FullAccess
    IAMFullAccess
    AmazonVPCFullAccess
{% endblockquote %}

AWS Console에서 IAM으로 들어가 kops라는 이름의 유저 그룹을 만들고 위 권한을 부여한 후 kops라는 이름의 사용자를 만들었습니다.

{% asset_img 2.JPG %}

사용자를 생성하면 **kops.csv** 라는 파일을 다운받을 수 있는데요, 이 파일을 잘 가지고 계셔야 합니다.
(아마 최초 1회 밖에 다운이 되지 않을겁니다.)

또한 EC2에 접근하기 위해 kops라는 이름의 키 페어도 만들었습니다.

{% asset_img 5.JPG %}

### 2.2. Bastion 서버 생성

처음에 저는 로컬 PC(Windows)에 kops와 kubectl 명령어를 설치하고 진행을 했습니다.
그런데 윈도우라 그런지 뭔가 진행이 매끄럽게 되지 않는 부분이 있었습니다.
(제 설정이 이상했을 수 있습니다.)
어쨌든 별 수 없이 명령어를 실행하고 접속하기 위한 Linux EC2 서버를 하나 생성해서 진행하니 매끄럽게 진행이 되었습니다.

{% blockquote %}
    사실 엄밀한 의미의 Bastion 서버는 아닙니다. 비용을 위해 네트워크를 public으로 세팅할 것이기 때문입니다.
{% endblockquote %}

대신 요금 절감을 위해 프리티어 사용이 가능한 EC2 인스턴스를 생성해서 사용하도록 하겠습니다.

{% asset_img 3.JPG %}

{% asset_img 4.JPG %}

키 페어는 위에서 만든 kops 키 페어를 선택해 생성합니다.

Amazon Linux EC2의 기본 유저는 `ec2-user`입니다. 생성 후 키 파일을 이용해 접속할 수 있습니다.

### 2.3. aws-cli, kops, kubectl 설치

먼저 kubectl 부터 설치하도록 하겠습니다.
과정은 {% link 문서 https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ %}에 나와 있는 내용과 동일합니다.

아래 명령어를 순차적으로 실행합니다.

{% codeblock install kubectl lang:Bash %}
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    kubectl version --client
{% endcodeblock %}

kubectl 사용시 편리함을 위해 자동완성을 위한 작업도 진행하겠습니다.({% link 문서 https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/ %})

{% codeblock set kubectl lang:Bash %}
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    kubectl version --client
{% endcodeblock %}

다음은 aws-cli입니다. {% link 문서 https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ %}를 참고했습니다.

{% codeblock install aws-cli lang:Bash %}
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
{% endcodeblock %}

aws-cli 설치 후 iam 계정을 연결하는 설정을 해야 합니다.
위에서 kops 계정을 만든 후 다운받은 **kops.csv**의 파일에서 내용을 확인해 아래 명령어를 실행합니다.

{% codeblock configure aws-cli lang:Bash %}
    aws configure
    AWS Access Key ID [None]: [csv파일에서 확인]
    AWS Secret Access Key [None]: [csv파일에서 확인]
    Default region name [None]: ap-northeast-2
    Default output format [None]: json
{% endcodeblock %}

kops 설치입니다.({% link 문서 https://kops.sigs.k8s.io/getting_started/install/#linux %})

{% codeblock install kops lang:Bash %}
    curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops
    sudo mv kops /usr/local/bin/kops
{% endcodeblock %}

## 3. DNS 세팅

{% link 문서 https://kops.sigs.k8s.io/getting_started/aws/#configure-dns %}에 따르면 kops를 이용해 쿠버네티스 클러스터를 구축하기 위해서는 도메인이 필요하다고 합니다.
route53에서 검색해보니 제가 사용할만한 적당한 도메인이 그렇게 비싸진 않아서(연 $12) 구매를 했습니다.

{% link 실제 도메인을 구매하지 않고 진행하는 방법 https://kops.sigs.k8s.io/gossip/ %}이 있다고 하는데 일단 저는 도메인 가격이 엄청 비싸다는 생각은 들지 않아서 구매 후 진행했습니다.

## 4. S3 bucket 생성

kops로 쿠버네티스를 사용하기 위해서는 S3 버킷이 필요합니다.
{% link 문서 https://kops.sigs.k8s.io/getting_started/aws/#cluster-state-storage %}에는 명령어로 버킷을 만드는 방법이 설명되어 있지만 저는 콘솔에서 만들도록 하겠습니다.

아래와 같이 세팅합니다.

- 리전 : 서울
- 퍼블릭 엑세스 차단
- 버전 관리 : 활성화
- 서버측 암호화 : 활성화 - Amazon S3 키(SSE-S3)

여기서 언급해야할 부분은 2가지입니다.

버전 관리는 문서에서 강하게 권장한다고 하니 설정했습니다.
서버측 암호화는 설정하지 않으면 SSE-S3와 함께 서버 측 AES256 버킷 암호화를 사용한다고 되어 있어서 설정했습니다.

## 5. Cluster 생성

### 5.1. 환경변수 세팅

Bastion 서버에 접속합니다.

명령어 사용시 자주 사용하기 때문에 아래처럼 /home/ec2-user/.bash_profile에 내용을 추가합니다.

{% codeblock 환경변수 추가 lang:Bash %}
    PATH=$PATH:$HOME/.local/bin:$HOME/bin
    KOPS_CLUSTER_NAME=[구매한 도메인]
    KOPS_STATE_STORE=s3://[S3 버킷 이름]

    export PATH KOPS_CLUSTER_NAME KOPS_STATE_STORE
{% endcodeblock %}

### 5.2. availability zone 확인

kops 명령어를 통해서 클러스터를 생성하기 위해서는 aws의 availability zone을 알아야 합니다. 서울 리전이라면 아래 명령어로 확인합니다.

{% codeblock availability zone 확인 lang:Bash %}
    aws ec2 describe-availability-zones --region ap-northeast-2
{% endcodeblock %}

명령어를 실행해서 나온 결과 중 State가 available인 경우 해당 존이 사용가능한 것 같습니다.
(저의 경우 전부 사용 가능한 것으로 나왔습니다.)

### 5.3. public key 세팅

저는 비용을 위해 네트워크를 모두 public 네트워크를 사용할 예정입니다.
(private 네트워크를 사용하면 NAT gateway를 사용하게 될 것이고, 제가 처음 EKS를 구축했을 때처럼 비용이 나가게 될겁니다.)

맨 처음 생성한 kops 키 페어를 /home/ec2-user/.ssh 하위에 두고 권한을 600으로 줍니다.
그리고 아래 명령어를 실행합니다.

{% codeblock create public key lang:Bash %}
    ssh-keygen -f ./kops.pem -y > kops.pub
{% endcodeblock %}

그리고 생성된 public key도 600 권한으로 변경합니다.

{% asset_img 7.JPG %}

### 5.4. 클러스터 생성

아래 명령어를 실행해 실제 쿠버네티스 클러스터를 생성합니다.
단, 디폴트 옵션으로 실행하면 제가 원하는 환경과 다른 경우가 생겨서 아래와 같이 실행했습니다.
옵션 종류와 디폴트 값에 대한 내용은 {% link 여기 https://kops.sigs.k8s.io/cli/kops_create_cluster/#options %}에서 확인할 수 있습니다.

{% codeblock set cluster lang:Bash %}
    kops create cluster \
    --name $KOPS_CLUSTER_NAME \
    --state $KOPS_STATE_STORE \
    --cloud aws \
    --zones ap-northeast-2a \
    --image ami-086a3601cd7a83590 \
    --master-zones ap-northeast-2a \
    --master-size t3a.medium \
    --master-volume-size 40 \
    --node-size t3a.medium \
    --node-volume-size 40 \
    --container-runtime docker \
    --ssh-public-key ./.ssh/kops.pub
{% endcodeblock %}

옵션 중 이미지 아이디는 (더 좋은 방법이 있을 것 같지만) 단순하게 EC2 생성화면에서 찾아서 사용했습니다.

{% asset_img 6.JPG %}

위 명령어는 단순히 클러스터를 어떤 환경으로 만들지에 대한 세팅을 하는 것입니다.
실제로 클러스터를 생성하기 위해서 아래 명령어를 추가적으로 실행합니다.

{% codeblock create cluster lang:Bash %}
    kops update cluster --name $KOPS_CLUSTER_NAME --yes --admin
{% endcodeblock %}

AWS console에 접속해 확인해보면 t3a.medium 으로 ec2 인스턴스가 2개 생성된 것을 확인할 수 있습니다.

또한 처음에 만든 key 파일을 이용해 생성된 ec2 인스턴스(master, worker node)에도 접속할 수 있습니다.

{% post_link kops-k8s-deploy-nginx 다음 포스트 %}에서는 추가적인 세팅 및 테스트를 해보도록 하겠습니다.

## ※ 비용

저와 같은 방식으로 쿠버네티스 클러스터를 생성하셨다면 한 달에 청구되는 비용은 아래와 같습니다.
(30일 기준)

- 도메인 : 연 $12
- Bastion 서버(t2.micro) : $10.368
- master(t3a.medium) : $33.696
- worker node(t3a.medium) : $33.696
- +Load Balancer, EBS, S3 등에서 청구되는 비용
- 총합 : 월 $77.76 + a

경험상 한 달에 약 $100 정도 청구되었습니다.
