---
categories:
  - 기타
tags:
  - elasticsearch
  - docker
date: 2022-07-17 23:14:02
title: Docker로 local Elasticsearch 띄우기
updated:
---

회사에서 Elasticsearch를 사용하고 있어서 공부의 필요성을 느끼던 차에 로컬에 공부용 es를 띄워보기로 했습니다.
저는 로컬에서 사용하는 환경들은 가능한한 docker로 사용하는 것을 선호합니다.
아무래도 이것 저것 설정을 바꾸었다가 새로 띄우고 하는 것들이 좀 용이하다고 생각되기 때문이죠.

# 1. Elasticsearch 설치

{% link docker hub https://hub.docker.com/_/elasticsearch %}에 검색해보니 역시나 Elasticsearch 이미지가 존재합니다.

그런데 pull을 받을 때부터 문제가 생겼습니다. `docker pull elasticsearch`를 실행하자 아래와 같은 메시지가 나옵니다.

{% blockquote %}
	Error response from daemon: manifest for elasticsearch:latest not found: manifest unknown: manifest unknown
{% endblockquote %}

찾아보니 `latest` 태그가 정상동작하지 않는 것 같습니다. {% link es 문서 https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html %}를 보니 버전을 명시하도록 되어 있더군요.
아래와 같이 pull을 받도록 하겠습니다.

{% codeblock lang:Bash %}
	docker pull docker.elastic.co/elasticsearch/elasticsearch:8.3.2
{% endcodeblock %}

이미지를 다 받았으면 문서에 나온대로 network를 생성합니다.

{% codeblock lang:Bash %}
	docker network create elastic
{% endcodeblock %}

이제 es를 실행하겠습니다.
사실 여기서 조금 헤맸었는데요, 블로그 글을 쓰려고 보니 {% link es 문서 https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html %}와 {% link docker hub https://hub.docker.com/_/elasticsearch %}에 나온 명령어가 조금 다르네요;;

{% codeblock ES문서의 실행문 lang:Bash %}
	docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.3.2
{% endcodeblock %}

{% codeblock Docker Hub의 실행문 lang:Bash %}
	docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
{% endcodeblock %}

세세한 부분들은 넘어가고 제일 다른 점은 `-e "discovery.type=single-node"`부분입니다.
저는 가상머신의 linux위에서 docker로 es를 띄웠는데 이때 메모리를 3기가로 세팅해뒀습니다. 해당 환경변수가 없으면 메모리를 늘리라는 식의 에러가 나게 됩니다.

구글링을 하던 도중 제가 하고자하는 것과 {% link 딱 맞는 글 https://levelup.gitconnected.com/how-to-run-elasticsearch-8-on-docker-for-local-development-401fd3fff829 %}을 하나 찾았습니다.
해당 글을 보고 여러 가지 옵션 세팅을 더할 수 있었는데요, 최종적인 실행문은 다음과 같습니다.

{% codeblock lang:Bash %}
	docker run \
	--name [CONTAINER_NAME] \
	-d \
	-p 9200:9200 \
	-p 9300:9300 \
	-e discovery.type=single-node \
	-e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
	-e xpack.security.enabled=false \
	--net elastic \
	docker.elastic.co/elasticsearch/elasticsearch:8.3.2
{% endcodeblock %}

각 환경 변수의 의미는 다음과 같습니다.
- discovery.type=single-node : 싱글 노드로 실행
- ES_JAVA_OPTS="-Xms1g -Xmx1g" : 최대, 최소 JVM heap 사이즈를 1GB로
- xpack.security.enabled=false : 인증 비활성화

위와 같이 실행 후 `localhost:9200`으로 요청을 날렸을 때 아래와 유사한 형태로 응답이 오면 성공입니다.

{% codeblock lang:JSON %}
	{
		"name": "75cb76c33fe5",
		"cluster_name": "docker-cluster",
		"cluster_uuid": "MU_ir5s6SWWXmvwjj6CpkQ",
		"version": {
			"number": "8.3.2",
			"build_type": "docker",
			"build_hash": "8b0b1f23fbebecc3c88e4464319dea8989f374fd",
			"build_date": "2022-07-06T15:15:15.901688194Z",
			"build_snapshot": false,
			"lucene_version": "9.2.0",
			"minimum_wire_compatibility_version": "7.17.0",
			"minimum_index_compatibility_version": "7.0.0"
		},
		"tagline": "You Know, for Search"
	}
{% endcodeblock %}

# 2. Kibana 띄우기

{% link 이 글 https://levelup.gitconnected.com/how-to-run-elasticsearch-8-on-docker-for-local-development-401fd3fff829 %}에 나와있는대로 Kibana까지 docker로 실행해보도록 하겠습니다.
포인트는 **ES 컨테이너를 띄울 때 사용한 network와 같은 network를 명시**하는 것과 가능한한 ES와 같은 버전을 사용하는 것입니다.

{% codeblock lang:Bash %}
	docker run \
	--name [CONTAINER_NAME] \
	--net elastic \
	-p 5601:5601 \
	docker.elastic.co/kibana/kibana:8.3.2
{% endcodeblock %}

어라? 그런데 뭔가 이상합니다. 로그를 보니 아래와 같은 메시지가 나옵니다.

{% blockquote %}
	i Kibana has not been configured.

	Go to http://0.0.0.0:5601/?code=617541 to get started.
{% endblockquote %}

브라우저로 해당 주소를 들어가보니 아래와 같이 Enrollment Token을 입력하라는 페이지가 나옵니다.

{% asset_img 1.png %}

분명 ES에서 인증을 껐으니까 이 페이지가 나오지 않아야 할 것 같은데 나옵니다.
마찬가지로 구글링을 하다가 우연히 방법을 찾았는데요, 아래와 같이 실행하시면 됩니다.

아래의 `Configure manually`를 클릭하면 아래와 같은 창이 나옵니다.

{% asset_img 2.png %}

제 경우는 docker가 docker desktop을 이용해 mac에 깔려있는게 아니라 VirtualBox를 이용해 띄운 linux 위에 docker가 떠있어서 그런지 localhost:9200으로 실행해도 정상적으로 작동하지 않았습니다.

대신 저 ip를 localhost가 아닌 docker가 사용하는 네트워크 ip를 사용하면 됩니다.
linux에서 `ip a` 명령어를 입력하고 나온 ip 중 **docker0**의 ip를 http로 입력합니다.

{% asset_img 3.png %}

이렇게 하면 secure하지 않다는 경고 메시지가 나오는데요, 어차피 로컬에서 연습용으로만 사용할테니 크게 상관없습니다.
`Configure Elastic`을 클릭합니다.

{% asset_img 4.png %}

그러면 Kibana까지 모두 정상적으로 세팅을 마칠 수 있게 됩니다.

# * Reference

- {% link Install Elasticsearch with Docker https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html %}
- {% link How to Run Elasticsearch 8 on Docker for Local Development https://levelup.gitconnected.com/how-to-run-elasticsearch-8-on-docker-for-local-development-401fd3fff829 %}