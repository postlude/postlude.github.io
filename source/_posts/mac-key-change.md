---
categories:
  - 기타
tags:
  - etc
date: 2022-02-06 17:59:51
title: Mac에서 키보드 세팅 변경하기
updated:
---

최근 이직을 하게 되면서 처음으로 맥을 사용하게 되었습니다.
(어떻게 보면 신기하게 보일 수도 있을 것 같습니다. 개발 일을 하면서 맥을 처음 쓴다는 게..)

그런데 키보드 배열이나 단축키 같은 것들이 윈도우와는 완전히 달라서 처음에 적응하는데 시간이 좀 걸렸습니다.
처음에는 맥에 적응을 할까도 생각해봤지만 이직을 해서 바쁜 와중에 단축키에 시간을 뺏기느니, 차라리 맥 세팅을 바꿔서 빠르게 사용할 수 있도록 하는게 낫겠다는 생각이 들었습니다.
(물론 이것도 전체 시간을 다 합치면 좀 걸리긴 했습니다.)

{% blockquote %}
	저는 맥을 무조건 저와 같은 방식으로 세팅해야 한다고 생각하지 않습니다.
	그냥 이런 방식으로 세팅하는 방법이 있다 정도로만 이해해주시면 될 것 같습니다.
{% endblockquote %}

## 0. 키 입력 확인

세팅을 하면서 키 입력을 확인하고 싶으시면 아래 사이트에서 확인이 가능합니다.

{% link Keyboard Checker https://keyboardchecker.com/ %}

## 1. Control / Command 변경

제일 처음에 한 작업은 Control 키와 Command 키를 변경한 것입니다.
기본 키 배열로 쓰시는 분들도 있을 수 있겠지만 저는 윈도우의 복사, 붙여넣기의 ctrl c, v가 너무 익숙해져서 변경했습니다.

[설정 - 키보드 - 보조 키]에서

{% asset_img 1.png %}

Control과 Command키를 서로 바꿔줍니다.
(저는 다른 세팅을 변경해서 다른 키보드로 나오는데 신경쓰지 말고 변경해주시면 됩니다.)

{% asset_img 2.png %}

## 2. 한/영 키 변경

맥에서 한/영 키는 caps lock과 같은 키를 씁니다. 저는 윈도우에 맞춰진 외부 키보드를 사용하기 때문에
이걸 윈도우에서 쓰던 한/영 키로 변경하도록 하겠습니다.

먼저 {% link 여기 https://karabiner-elements.pqrs.org/ %}에서 Karabiner를 설치합니다.

그 후 Preferences에서 아래와 같이 세팅합니다.

{% asset_img 3.png %}

이 세팅의 목적은 right option 키를 존재하지 않는 다른 키로 맵핑하기 위함입니다.
(다만 이유는 알 수 없으나 F17이 아닌 다른 키로 맵핑했을 때는 정상적으로 동작하지 않더군요.)

이후 [설정 - 키보드 - 단축키]에서 `이전 입력 소스 선택`을 F17로 변경하시면 키보드의 한/영 키를 사용할 수 있게 됩니다.

{% asset_img 4.png %}

추가적으로 한/영 키를 사용할 수 있게 되었으니 caps lock키를 이용한 한/영 변경은 필요가 없어졌기 때문에
[설정 - 키보드 - 입력 소스]에서 해제하도록 하겠습니다.

{% asset_img 5.png %}

## 3. 추가 키 맵핑

위에서 설치한 Karabiner를 통해 추가적으로 다른 키 맵핑들을 세팅할 수 있습니다.

[Preferences - Complex modifications]에서 `Add rule`을 통해 키를 맵핑할 수 있습니다.

{% asset_img 6.png %}

제가 추가한 것은 위 스크린샷에 있는 4가지이며 각각의 내용은 다음과 같습니다.

- 첫 번째 : ctrl + 방향키로 띄어쓰기 단위 커서 이동 가능
- 두 번째 : ctrl + 백스페이스 키로 띄어쓰기 단위 문자 삭제 가능
- 세 번째 : ctrl + shift + 방향키로 띄어쓰기 단위 문자 선택 가능
- 네 번째 : finder에서 F2키로 이름 변경 가능

rule에 관한 내용은 {% link 여기 https://ke-complex-modifications.pqrs.org/ %}에서 확인하실 수 있습니다.
다만, 적용하실 때는 하나씩 직접 해보고 원하는 작동 여부를 확인하시기 바랍니다.
제가 했을때는 rule이 추가되어도 동작하지 않는 것들도 많았습니다.

## 4. 크롬 새로고침

크롬에서 새로고침을 할 때 윈도우처럼 F5키로 할 수 있도록 세팅하는 방법입니다.

[설정 - 키보드 - 단축키 - 앱 단축키]에서 `+`키를 눌러 아래와 같이 추가해줍니다.

{% asset_img 7.png %}

이런 방식이면 다른 단축키도 세팅할 수 있을 것 같은데 더 찾지는 못했습니다. 아시는 분들은 댓글 부탁드리겠습니다.
(탭 이동을 ctrl + page up/down으로 변경하고 싶은데 말이죠..)

## 5. 터미널 세팅

저 같은 경우는 터미널에서 사용하는 단축키가 리눅스 단축키에 익숙해져 있습니다.
그래서 맥 기본 터미널인 `zsh`의 단축키를 변경하고 싶었는데 방법을 찾지 못했습니다.

제가 찾은 방법은 `iTerm2`를 설치해서 키 세팅을 변경하는 방법입니다.

### 5.1. iTerm2 설치

{% link 공식 홈페이지 https://iterm2.com/index.html %}에서 다운받아 설치합니다.

### 5.2. 키 맵핑

iTerm2를 실행한 후 `ctrl(command) + ,`를 이용해 설정을 엽니다.
(대부분의 맥 프로그램들은 `command + ,`가 설정을 여는 단축키라고 합니다.)

Keys에 있는 항목들을 통해 원하는 형태로 키 맵핑을 할 수 있습니다.
원하시는데로 세팅하시면 되는데 저는 아래와 같이 세팅했습니다.

- `ctrl + 방향키`로 단어 단위 커서 이동
- `ctrl + 백스페이스`로 단어 단위 문자 삭제
- 여러 개의 탭이 있을 경우 `ctrl + page up/down`으로 이동
- `home / end` 키를 이용해 커서를 문장 처음 / 끝으로 이동
- `ctrl + k`로 커서 이후 문자 삭제
- `ctrl + u`로 현재 입력 모두 삭제(이건 리눅스 명령어와 조금 동작이 다릅니다.)

{% asset_img 8.png %}

{% asset_img 9.png %}

관련해서는 링크를 남겨드릴테니 참고하시기 바랍니다.

- {% link iTerm2와 zsh로 깔끔한 Mac 터미널 환경 만들기 https://tutorialpost.apptilus.com/code/posts/tools/mac-cli-with-iterm2-zsh/ %}
- {% link 5 must-have key mappings on iTerm2 to be more productive https://medium.com/macoclock/5-must-have-key-mappings-on-iterm2-to-be-more-productive-21c4daf56348 %}
- {% link iTerm key bindings https://gist.github.com/ezekg/ed57b777b859975084b4 %}

## 6. 마치며

사실 아직 좀 아쉬운 점이 몇 가지 있습니다.
크롬 탭 이동이라던가 finder에서 delete로 삭제하기, 터미널에서 ctrl + l로 커서 올리기 등등.

변경할 수 있는 방법을 찾으면 그 때 다시 포스팅을 올리도록 하겠습니다.