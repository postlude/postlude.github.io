---
categories:
  - 기타
tags:
  - job
date: 2019-06-16 00:00:00
title: 스타트업 6개월 후기
updated:
---

제가 스타트업에서 개발자로 일을 한지 벌써 6개월이라는 시간이 흘렀네요.
스스로를 점검할 겸 느낀 점들을 적어볼까 합니다.

# 1. git... GIT..!!

입사 초반에 절실하게 느낀 것은 git에 대한 내용입니다.
개발 회사에서 버전 관리 툴을 쓰지 않는 곳은 거의 없을 것이라고 생각하는데요,
그 git을 잘 활용할 수 있어야 되겠다는 것을 피부로 많이 느끼게 되었습니다.

개인적으로 github 계정을 가지고 나름 제가 짠 코드를 올리고 있었는데
그런 부분들이 많이 도움이 되었습니다. 기본적인 명령어들을 알고 있었기 때문이죠.
더불어 제가 개발자로 취업을 준비하시는 분들에게 공부를 권장하고 싶은 것은 **stash**에 관한 내용입니다.

저만 그럴지는 모르겠으나 저는 **git stash**를 정~말 많이 사용합니다.
그래서 입사 후 얼마 되지 않았을 때 블로그에 {% post_link git-stash 정리하는 글 %}을 올리기도 했고요.

또한 저희 회사는 git flow 를 사용하는데 기본적으로 git 명령어를 사용하는 것과 크게 다르지 않기 때문에 적응하는데도 큰 무리가 없었습니다.


# 2. 평일에도 힘들지만 공부해야 한다.

이건 사람마다 공부 패턴이 다르기 때문에 모두 이렇게 해야되는 것은 아닙니다만,
하나는 확실합니다.

{% blockquote %}
    정말 꾸준히 공부해야합니다.
{% endblockquote %}

그 이유를 최근에 명확하게 알게 되었습니다. 바로 **언제 바빠질지 모르기 모르기 때문**입니다.
제 github 활동을 한 번 보겠습니다.

{% asset_img 1.JPG %}

...정말 부끄럽기 짝이 없네요.
변명을 하자면 5월부터 회사 일이 점점 바빠지기 시작하더니 최근 몇 주간은 너무 바빠서 주말 출근도 할 지경이었습니다.
(물론 그 전에는 그냥 게으른 것이었지만..)

어쨌든, 제가 공부를 조금씩 미루면서 스스로에게 한 변명은 이것이었습니다.

'내일하지 뭐. 나중에 하지 뭐.'

그런데 그게 회사 일이 바빠지니까 **아예 공부할 시간조차 나지 않더군요.**
그러니 평소에 조금씩이라도 꾸준히 공부해야 된다는 것을 최근 일로 많이 느끼게 되었습니다.

# 3. 무조건은 없다.

개발을 하면서 경계해야되는 부분이죠. 세상에 '무조건' 이라는 것은 없다.
그럼에도 불구하고 공부를 하면서 익혔던 부분들을 무의식 중에 적용을 하다가 낭패(?)를 겪은 적이 있습니다.

회사에서 RDB 접속을 위해 Workbench를 사용하고 있습니다.
DB라는 것이 항상 신중하게 사용해야되기 때문에 저는 Workbench의 auto commit 기능을 꺼놓고 사용을 하고 있었습니다.
그런데 제가 로컬에서 작성한 api로 데이터를 넣은 후 Workbench에서 select문으로 확인하면 데이터가 들어가지 않은 것으로 표시가 되었습니다.
계속 헤매다가 다른 분이 도와주셔서 찾은 원인은 바로 그 auto commit 때문이었습니다.

사실 데이터는 제대로 들어갔는데, auto commit이 꺼져있는 상태에서 select 문을 돌리니까
마치 dirty read 처럼 데이터가 들어가기 전 내용만 불러오는 현상이 있었습니다.

저는 툴 사용에 있어서 'auto commit 은 위험하니까 무조건 끄고 사용해야된다' 라는 생각을 가지고 있었는데
겪어보니 '무조건' 이라는 건 역시나 존재하지 않더군요.

# 4. 쓸데없는 경력이란 없다.

저는 지금 회사에 들어오기 전에는 개발자가 아닌 WAS 엔지니어로 일을 했었습니다.
개발이 너무 하고 싶었기에 그 회사를 그만두고 개발자로서 취업 준비를 하고 지금 회사에 들어오게 되었습니다.

이력서에 제 경력을 쓰기는 했습니다만, 실무적으로 크게 도움이 될 것이라고는 생각하지 않았습니다.
그런데 이게 왠걸, 도움이 되더군요.

이전 회사에서 PaaS를 조금 접할 수 있는 기회가 있었고, 덕분에 Docker에 대해 알게 되었었는데
지금 회사에서도 Docker를 사용하고 있어서 상당히 도움이 많이 되었습니다.

완전히 다른 분야에서 일을 한 것이 아니라면, **정말로 쓸모없는 경력은 없습니다.**
단, 한 가지 조건은 있겠네요. 바로,

{% blockquote %}
    직장에서 무언가 얻는 것이 꾸준히 있어야 한다는 점입니다.
{% endblockquote %}

이전 직장은 신입사원이 들어온다고 해서 차분하게 무언가를 알려주거나 교육해주는 시스템은 없었습니다.
(물론 중소기업에서 그런게 있는데가 많지는 않겠지만요.)

하지만 그렇다고 제가 스스로 공부도 하지 않고 배우려고 하지 않았다면, 전 직장에서의 경력이 정말로 아무 의미가 없는 시간이 되었을지도 모릅니다.
그리고 지금 회사에서 도움이 될 법한 내용들을 알 수 없었을지도 모릅니다.

그러니 회사가 좋든 나쁘든 어떤 곳을 다니는 동안에는 스스로 얻어가는 것이 있어야 할 것 같습니다.
월급 이외에 것을 말이죠.
그게 지식적인 것이 될수도 있고, 다른 것이 될수도 있겠지만 뭐가 되든 성장한다고 느끼고 있으면 될 것 같습니다.

그래야 나중에 다른 직장을 가더라도 도움이 될테니까요.

# 5. 개발 외적인 지식이 쓰일데도 있다.

이건 그냥 번외 이야기로 제가 겪은 경험을 조금 써보고자 합니다.

최근에 업무로 한 작업 중 일종의 통계를 내는 작업을 한 적이 있습니다.
DB에서 해당 내용들을 조회를 해서 결과 값을 계산하는 작업이었습니다.

업무를 받았을 때는 단순히 모든 데이터들의 값들을 하나씩 다 조회해서 전부 계산을 하는 식으로 전달을 받았었는데,
이걸 천천히 살펴보다보니 다른 방식으로 계산이 가능할 것 같았습니다.

결과적으로 얻고자 하는 값을 풀어서 보니
모든 데이터들을 전부 탐색하는 것이 아니라 전체를 일괄적으로 탐색해서 나온 값을 넣어서도 계산이 가능했습니다.

**즉, 아주 간단한 수식 변환으로 DB 탐색의 횟수를 확 줄일 수 있었습니다.**

이처럼 개발 관련 지식이 아니더라도 업무에 도움되는 경우가 있었습니다.
개인적으로는 꽤 신선한 경험이었습니다.

# 6. 마무리

6개월이라는 시간동안 제가 확실히 느낀 것은

**'역시 개발이 내 적성에 맞구나'**

라는 것이었습니다.
앞으로 6개월의 시간이 더 흘러서 1년의 시간이 지났을 때는
조금 더 성장한 모습으로 후기를 쓸 수 있으면 좋겠네요.