---
categories:
  - 기타
tags:
  - linux
date: 2020-03-09 23:08:45
title: VirtualBox에 CentOS 7 minimal로 설치하기
updated:
---


{% post_link virtualbox-centos-install 예전 포스트 %}에서 virtualbox에 CentOS7을 설치하는 것을 포스팅한 적이 있습니다.

솔직히 말하자면 해당 포스트에서 CentOS에 gui를 설치한 것은 다음과 같은 이유였습니다.

1. MobaXterm을 알기 전이라 포트 포워딩으로 사용할만한 적절한 툴을 알지 못했습니다.
2. minimal로 설치했을 때 공유 디렉토리를 사용하기 위한 설정이 자꾸 실패했습니다.

테스트를 하던 도중 드디어 CentOS minimal에 공유 폴더 설정(VBox 게스트 확장 설치)에 성공했습니다.

조금 더 정리된 VBox CentOS 7 설치 포스트라고 봐주시면 될 것 같습니다.

{% blockquote %}
    포스트 내용은 제가 오류를 범한 프로세스를 그대로 따라 갑니다.
    한 번에 하시고 싶으신 분들은 맨 마지막 정리 부분만 읽어주시면 됩니다.
{% endblockquote %}

# 1. CentOS 설치

이 부분은 이전과 거의 동일합니다. {% post_link virtualbox-centos-install 이전 포스트 %}를 참고하시면 됩니다.
다른 부분은 다음과 같습니다.

1. Extension Pack은 필요하지 않습니다.
어차피 포트 포워딩 후 MobaXterm을 사용할 예정이므로 창 크기 조절에 신경쓸 필요가 없습니다.

2. SOFTWARE SELECTION에서 Minimal Install로 선택 후 설치하시면 됩니다.

{% asset_img 1.JPG %}

# 2. 기본 설정

설정은 이전과 크게 다르지 않습니다. 다만, MobaXterm을 사용하기 위해 포트포워딩 설정을 해주도록 합니다.

호스트 포트는 1023 이하가 아니라면 크게 중요하지 않습니다. ssh로 접속해야 하므로 게스트 포트는 22번으로 설정합니다.

{% asset_img 2.JPG %}

{% asset_img 3.JPG %}

여기까지만 설정하면 사용하는데 큰 지장이 없습니다. 딱 한 가지, 공유 폴더를 제외하면 말이죠.

일단, 이전에 했던데로 VBox에서 공유 폴더 설정도 미리 해주도록 하겠습니다.

{% asset_img 4.JPG %}

# 3. VirtualBox 게스트 확장 설치

## 3.1 마운트

GUI 설치 버전에서 했던 것과 동일하게 설치를 시도하겠습니다.

장치 - 게스트 확장 CD 이미지 삽입

{% asset_img 5.jpg %}

이전에는 CD 삽입만 해도 설치 화면이 나왔지만 minimal 설치에서는 아무런 반응이 없습니다.

직접 마운트를 시킨 후에 설치를 해야합니다.

우선 마운트시킬 디렉토리를 `/mnt` 하위에 `cdrom` 이라는 이름으로 생성 후 `/dev/cdrom`을 마운트 했습니다.

{% codeblock %}
    mkdir /mnt/cdrom
    mount /dev/cdrom /mnt/cdrom
{% endcodeblock %}

그러면 아래와 같이 exe 파일과 sh 파일이 보이게 됩니다.

{% asset_img 6.JPG %}

## 3.2 패키지 설치

여기서 우리가 실행해야할 것은 VBoxLinuxAdditions.run 입니다.

{% codeblock %}
    ./VBoxLinuxAdditions.run
{% endcodeblock %}

{% asset_img 7.JPG %}

bzip2 이라는 패키지가 없다고 나오네요. yum으로 설치하도록 하겠습니다.

{% codeblock %}
    yum install -y bzip2
{% endcodeblock %}

설치 후 다시 VBoxLinuxAdditions.run 을 실행합니다.

{% asset_img 8.JPG %}

이번에는 조금 더 실행이 됐지만 fail이 났습니다.

메시지를 보면 이유를 알 수 있는데, target kernel 버전 3.10.0-1062.el7.x86_64에 맞는 kernel header가 없다고 합니다.

{% asset_img 9.JPG %}

위에서 보이는 것과 같이 kernel-headers라는 패키지가 설치되어 있지 않습니다. 설치하도록 하겠습니다.

{% codeblock %}
    yum install -y kernel-headers
{% endcodeblock %}

다시 VBoxLinuxAdditions.run 을 실행하도록 하겠습니다.

{% asset_img 10.JPG %}

그런데 또 다시 동일한 에러가 납니다. 확인해보겠습니다.

{% asset_img 11.JPG %}

설치된 kernel(3.10.0-1062.el7) 버전과 kernel-headers(3.10.0-1062.18.1.el7) 버전이 다릅니다.

이 버전을 맞춰줘야 합니다.

kernel을 업데이트 하고 완료되면 재부팅하도록 합니다.

{% codeblock %}
    yum update -y kernel
    reboot
{% endcodeblock %}

{% asset_img 12.JPG %}

재부팅을 하면 선택화면에서 기존에는 2개였던 것이 3개로 선택지가 늘어난 것을 알 수 있습니다.

자세히 보면 하나는 이전 버전의 커널이고, 다른 하나는 업데이트 후 설치된 버전의 커널입니다.

**신규 버전**으로 부팅합니다.

아래의 명령어를 통해서도 현재 커널 버전을 알 수 있습니다.

{% codeblock %}
    uname -r
{% endcodeblock %}

{% asset_img 13.JPG %}

신규 버전으로 잘 부팅되었습니다. 이제 다시 VBoxLinuxAdditions.run 을 실행해야 하는데 재부팅했으므로 다시 마운트를 시켜야 합니다.

{% codeblock %}
    cd /mnt
    mount /dev/cdrom ./cdrom
    cd cdrom
    ./VBoxLinuxAdditions.run
{% endcodeblock %}

다시 실행했는데도 이게 왠걸, 동일한 에러가 납니다. 구글링을 해봤더니 아래와 같은 내용이 나옵니다.

{% blockquote https://forums.virtualbox.org/viewtopic.php?f=3&t=91563 %}
    I suspect the version of kernel-devel does not match the version of your running kernel
{% endblockquote %}

kernel-devel을 설치하도록 하겠습니다.

{% codeblock %}
    yum install -y kernel-devel
{% endcodeblock %}

VBoxLinuxAdditions.run 을 다시 실행합니다.

{% asset_img 14.JPG %}

이번엔 에러 메시지가 조금 달라졌습니다. 몇 가지 필요한 패키지들이 없다고 하네요. 설치하겠습니다.

{% codeblock %}
    yum install -y gcc make perl
{% endcodeblock %}

VBoxLinuxAdditions.run 을 다시 실행하면

{% asset_img 15.JPG %}

드디어! 정상적으로 설치된 것을 볼 수 있습니다.

게스트 확장 설치가 완료되었기 때문에 `/media` 경로로 가면 이전에 설정한 공유 폴더가 보이게 됩니다.

{% asset_img 16.JPG %}

## 3.3 이전 버전 커널 삭제

이제부터는 새롭게 업데이트한 버전의 커널을 사용할 것이므로 이전 버전은 필요하지 않습니다.

따라서 **yum remove** 나 **package-cleanup** 명령어를 통해 이전 버전 커널을 삭제하도록 하겠습니다.
(단, package-cleanup 명령어를 사용하기 위해서는 yum-utils 패키지가 설치되어 있어야 합니다.)

{% codeblock %}
    yum remove kernel.x86_64-3.10.0-1062.el7
{% endcodeblock %}

{% codeblock %}
    yum install -y yum-utils
    package-cleanup --oldkernels --count=1
{% endcodeblock %}

{% link 참고 https://linuxconfig.org/how-to-remove-old-unused-kernels-on-centos-linux %}

# 4. 정리

정리하자면 아래의 프로세스로 진행됩니다.

1. CentOS 7 minimal 설치

2. 패키지 설치
{% codeblock %}
    yum install -y bzip2 kernel-headers kernel-devel gcc make perl
{% endcodeblock %}

3. 커널 업데이트 후 재부팅 - 재부팅시 업데이트한 버전의 커널 선택
   (보통 제일 위에 것일 겁니다.)
{% codeblock %}
    yum update -y kernel
{% endcodeblock %}

4. 장치 - 게스트 확장 CD 이미지 삽입

5. 마운트 후 VBoxLinuxAdditions.run 실행
{% codeblock %}
    mkdir /mnt/cdrom
    cd /mnt
    mount /dev/cdrom ./cdrom
    cd cdrom
    ./VBoxLinuxAdditions.run
{% endcodeblock %}

6. 이전 버전 커널 삭제
{% codeblock %}
    yum install -y yum-utils
    package-cleanup --oldkernels --count=1
{% endcodeblock %}