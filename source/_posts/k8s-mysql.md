---
categories:
  - K8S
tags:
  - k8s
  - db
date: 2021-08-10 16:56:03
title: Kubernetes에 MySQL Pod 띄우기
updated:
---

이번 포스트에서는 kubernetes 위에 mysql pod를 띄우는 일을 정리해보도록 하겠습니다.

## 1. 기본 Deployment 작성

처음에는 가장 기본적인 deployment를 작성해보도록 하겠습니다. 아래와 같이 작성하면 됩니다.

{% codeblock deployment.yaml lang:yaml %}
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mysql
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - name: mysql
            image: mysql:8.0.26
            env:
            - name: MYSQL_ROOT_PASSWORD
              value: root
            ports:
            - containerPort: 3306
{% endcodeblock %}

{% codeblock apply deployment.yaml lang:Bash %}
    kubectl apply -f deployment.yaml
{% endcodeblock %}

여기서 중요한 것은 `MYSQL_ROOT_PASSWORD`입니다.
mysql을 실행시킬 때 `MYSQL_ROOT_PASSWORD`을 세팅하지 않으면 pod가 정상적으로 뜨지 않게 됩니다. 따라서 환경변수를 세팅하도록 합니다.

이렇게 하면 dockerhub로 부터 mysql 이미지를 pull 받아 pod를 띄우게 됩니다. 

## 2. Secret 사용

루트 패스워드를 세팅한 것은 좋은데 이렇게 되면 deployment.yaml에 패스워드를 그대로 노출하게 됩니다.
이것보다는 kubernetes의 secret을 이용하는 것이 보안적으로 더 좋은 방법이므로 사용해보도록 하겠습니다.

기본적으로 secret은 아래와 같은 형태로 만들면 됩니다.

{% codeblock secret.yaml lang:yaml %}
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysql-root
    type: Opaque
    data:
      password: [ROOT PASSWORD]
{% endcodeblock %}

data 하위에 key-value 형태로 원하는 값을 넣으면 됩니다. 저는 password라는 이름으로 사용할 비밀번호를 작성했습니다.
여기서 중요한 점은 secret에 넣는 value 값들은 **base64로 인코딩한 값**을 넣어야 한다는 것입니다.
예를 들어 패스워드를 `root`로 사용한다고 하면 `password: root`로 작성하는 것이 아니라 `password: [root를 base64로 인코딩한 문자열]`로 작성하셔야 합니다.

base64로 인코딩한 문자열은 리눅스의 echo 를 통해서 아래와 같이 알아낼 수 있습니다.

{% codeblock encoding base64 lang:Bash %}
    echo -n root | base64
{% endcodeblock %}

여기서 중요한 점은 **-n 옵션**입니다.
echo 명령어는 자동으로 trailing newline을 삽입한다고 합니다. **따라서 -n 옵션으로 trailing newline을 없앤 후에 인코딩을 해야 정상적으로 인코딩된 문자열을 얻을 수 있습니다.**
실제로 -n 있는 상태에서 인코딩된 문자열은 `cm9vdA==` 이고, -n 옵션이 없는 상태에서 인코딩된 문자열은 `cm9vdAo=` 으로 서로 다른 것을 알 수 있습니다.

아래와 같은 secret을 만들고 deployment.yaml도 수정하도록 하겠습니다.

{% codeblock secret.yaml lang:yaml %}
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysql-root
    type: Opaque
    data:
      password: cm9vdA==
{% endcodeblock %}

{% codeblock deployment.yaml lang:yaml %}
    ...
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-root
          key: password
    ...
{% endcodeblock %}

만약 -n 옵션이 없는 상태에서 인코딩된 문자열을 사용하면 어떻게 될까요?
secret과 deployment 모두 정상적으로 생성이 되지만 mysql pod가 뜨면서 **mysql: unknown option '--"'** 과 같은 에러가 납니다. 그리고 pod가 계속해서 생성되었다가 종료되었다가를 반복하게 됩니다.
실제로 제가 이런 실수를 해서 원인을 한참을 찾았었습니다.
다행히 {% link 이 글 https://stackoverflow.com/questions/62985541/mysql-unknown-option-in-kubernetes %}을 보고 원인을 찾아 해결할 수 있었는데요, 저와 같은 실수를 하지 않으시길 바라겠습니다.

## 3. Volume 사용하기

지금까지 작업을 통해 띄운 mysql pod는 volume을 사용하지 않고 있기 때문에 만약 pod 종료되었다가 다시 시작되면 그 동안 DB에 저장된 모든 내용은 날아가게 됩니다.
따라서 PV, PVC를 이용해 volume을 연결해주도록 하겠습니다.

저는 kubernetes를 kops를 이용해 EC2 서버 위에 구동하고 있기 때문에 EBS 볼륨을 생성하고 그 볼륨을 연결하려고 했지만... 볼륨을 사용하는 것 자체가 비용이 추가가 되기 때문에 node의 특정 디렉토리를 사용하는 volume(**local-storage**)을 만들어 사용하기로 했습니다.

먼저 node로 사용 중인 ec2에 접속해 `/volume/pv1` 디렉토리를 만들었습니다. 또한 pv가 특정 node에 연결된 것을 세팅하기 위해 node에 label을 추가했습니다.
저 같은 경우 `name=node1`로 label을 추가했습니다.

{% codeblock add label lang:Bash %}
    kubectl label nodes [NODE_NAME] name=node1
{% endcodeblock %}

그리고 아래와 같이 pv, pvc를 만들어주었습니다.

{% codeblock pv.yaml lang:yaml %}
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv1
    spec:
      capacity:
        storage: 10Gi
      volumeMode: Filesystem
      accessModes:
      - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      storageClassName: local-storage
      local:
        path: /volume/pv1
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - key: name
              operator: In
              values:
              - node1
{% endcodeblock %}

{% codeblock pv.yaml lang:yaml %}
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc1
    spec:
      storageClassName: local-storage
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
{% endcodeblock %}

그리고 deployment.yaml을 아래와 같이 수정하도록 하겠습니다.

{% codeblock deployment.yaml lang:yaml %}
    ...
    spec:
      containers:
      - name: mysql
        image: mysql:8.0.26
        volumeMounts:
        - name: volume1
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root
              key: password
        ports:
        - containerPort: 3306
      volumes:
      - name: volume1
        persistentVolumeClaim:
          claimName: pvc1
    ...
{% endcodeblock %}

pod가 재시작된 후 node의 `/volume/pv1` 디렉토리에 데이터들이 쌓이면 정상적으로 연결된 것입니다.

## ※ 참고 사항

application 서버에서 mysql pod에 접속하기 위해서는 mysql pod와 동일한 네임스페이스에 pod를 띄운 후 mysql 서비스 이름과 port로 접속이 가능합니다.

## ※ 참고 링크

- {% link kubernetes docs https://kubernetes.io/docs/concepts/storage/volumes/#local %}
- {% link Kubernetes에서 Local Persistent Volume을 사용하여 local disk 사용 https://lapee79.github.io/article/use-a-local-disk-by-local-volume-static-provisioner-in-kubernetes/ %}