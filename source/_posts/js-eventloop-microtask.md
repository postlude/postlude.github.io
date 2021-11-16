---
categories:
  - JavaScript
tags:
  - javascript
date: 2021-11-16 17:06:37
title: JS의 이벤트 루프, Task, Microtask
updated:
---


{% post_link javascript-execution-context 지난 포스트 %}에서 JS의 실행 컨텍스트(Execution Context)에 대해 정리했습니다.
이번에는 이벤트 루프와 Task, Micro Task에 대해 정리해보고자 합니다.

## 1. 이벤트 루프(Event Loop)

이벤트 루프에 관해서 이 영상보다 더 좋은 설명은 보지 못한 것 같습니다.
한글 자막도 잘되어 있으니 꼭 시청해보시는 걸 권장합니다.

{% youtube 8aGhZQkoFbQ %}

영상에 나온 내용과 기타 구글링을 통해 찾은 내용들을 정리하면 다음과 같습니다.

- JS 엔진은 이벤트 루프가 없다.
- 이벤트 루프와 WebAPI(Node.js의 경우 C++ API) 등은 브라우저(혹은 Node.js)에서 제공한다.
- 이벤트 루프는 call stack이 비어있으면 task queue에서 task를 하나 꺼내서 stack에 쌓아 실행될 수 있도록 한다.
- JS가 single thread라는 말은 곧 **call stack이 하나다**라는 말과 같다.

## 2. Task Queue vs. Microtask Queue

Task는 아래와 같은 작업을 말합니다. 흔히 우리가 알고 있는 JS 작업들입니다.

- 스크립트 실행
- 이벤트
- HTML 파싱
- 콜백
- Fetch, Ajax
- DOM 조작

위 작업들은 `task queue`에 쌓여 있다가 이벤트 루프에 의해 stack이 비어 있을 때 stack으로 올라가게 됩니다.

반면에 Microtask는 task queue가 아닌 `microtask queue(job queue)`라는 별개의 queue에 쌓였다가 이벤트 루프에 의해 call stack으로 올라가게 됩니다.

이 microtask에 해당하는 것 중 하나가 바로 **Promise**입니다.
(그 외에도 process.nextTick, Object.observe, MutationObserver가 있다고 합니다.)

그리고 microtask가 task보다 높은 우선 순위를 가지고 있습니다.

## 3. 예시

아래의 코드를 예시로 들어보겠습니다.

{% codeblock lang:JavaScript %}
    console.log('start');

    setTimeout(() => {
        console.log('settimeout 1');
    }, 0);

    Promise.resolve()
        .then(() => {
            console.log('promise 1');
        })
        .then(() => {
            console.log('promise 2');
        });

    setTimeout(() => {
        console.log('settimeout 2');
    }, 0);

    console.log('end');

    // 실행결과
    // start
    // end
    // promise 1
    // promise 2
    // settimeout 1
    // settimeout 2
{% endcodeblock %}

setTimeout과 Promise의 콜백으로 전달된 함수들은 각각 `task queue`와 `micro task queue`에 쌓이게 됩니다.
start, end가 출력된 후 이벤트 루프는 비어있는 call stack에 올릴 task를 결정하게 되는데 micro task의 우선 순위가 더 높기 때문에 promise 1, 2가 먼저 출력되게 됩니다.

## ※ 참고 링크

- {% link [Javascript] 자바스크립트의 호출 스택과 이벤트 루프 https://iamsjy17.github.io/javascript/2019/07/20/how-to-works-js.html %}
- {% link 비동기와 Promise #3 https://iamsjy17.github.io/javascript/2019/07/20/how-to-works-js.html %}
- {% link [JavaScript] Task Queue말고 다른 큐가 더 있다고? (MicroTask Queue, Animation Frames) https://velog.io/@titu/JavaScript-Task-Queue%EB%A7%90%EA%B3%A0-%EB%8B%A4%EB%A5%B8-%ED%81%90%EA%B0%80-%EB%8D%94-%EC%9E%88%EB%8B%A4%EA%B3%A0-MicroTask-Queue-Animation-Frames-Render-Queue %}