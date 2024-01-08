---
categories:
  - Git
tags:
  - git
date: 2020-08-11 22:50:27
title: git checkout 에러 - cannot create directory Permission denied
updated:
---


얼마 전 개인적으로 node 프로젝트를 진행하던 도중 겪었던 일입니다.

# 1. 증상

1. git status에서는 분명히 클린한 상태인데 checkout을 하면 에러가 납니다.
2. 에러 메시지는 아래와 유사한 형태로 나옵니다.

{% blockquote %}
    cannot create directory: Permission denied
{% endblockquote %}

3. 이후 git status로 상태를 보면 마치 checkout을 시도했던 브랜치의 커밋이
checkout으로 변경하려고 했던 브랜치로 **강제로 merge되려다 실패한 것처럼** 보입니다.
(없던 파일이나 디렉토리를 생성하려다 실패한 것처럼 보입니다.)

# 2. 검색

열심히 구글링을 해서 저와 비슷한 현상을 겪은 사람을 찾았습니다.

{% link stackoverflow 링크 https://stackoverflow.com/questions/39650678/git-checkout-error-cannot-create-directory-permission-denied %}

여기에 달린 답변들이 여러 가지가 있었습니다.

{% blockquote stackoverflow https://stackoverflow.com/questions/39650678/git-checkout-error-cannot-create-directory-permission-denied %}
    1. git reset --hard 를 사용해라
    2. 어딘가에서 파일을 사용 중일 것이다. 그러므로 vscode를 전부 종료하고 시도해봐라
    3. (위와 같은 이유로) 작업 관리자에서 관련있는 프로세스를 강제 종료해라
{% endblockquote %}

그런데 전 이 방법을 모두 시도해봤지만 해결되지 않았습니다.

# 3. 미궁속으로..

여러 가지 방법을 시도 중 **설마 정말로 권한 문제일까** 싶어서 vscode를 관리자 권한으로 실행시키니 해결이 되는 것 같았습니다.

문제는 checkout이 한 번은 되는데 한 번 더 시도하면 똑같은 현상이 발생한다는 것이었지요.

# 4. 해결

그러던 중 다음과 같은 프로그램을 찾았습니다.

{% link process explorer https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer %}

링크에 들어가보시면 아시겠지만, 윈도우에서 각종 프로세스의 상태를 볼 수 있는 프로그램입니다.
(MS 홈페이지에 있는 것이니 믿고 사용해도 되지 않을까 싶습니다.)

어쨌든, 해당 프로그램을 설치하고 실행해보니 떡하고 node 프로그램이 떠있는 것을 발견했습니다..

vscode를 모두 종료한 상태였는데도 말이죠.

해당 프로세스를 강제로 종료시킨 후 checkout을 시도해보니 너무나 깔끔하게 잘 실행되었습니다.

# 5. 결론

아주 드물게 node 프로세스가 정상적으로 종료되지 않는 경우가 발생한 것이 아닌가 싶습니다.
(사실 node의 문제인지 pm2 문제인지 vscode 문제인지는 잘 모르겠습니다.)

뭐, 덕분에 유용한 프로그램을 알게 됐으니 좋게 생각해야겠네요..
