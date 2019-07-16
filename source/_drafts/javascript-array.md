---
categories:
  - null
tags:
  - null
date: 2019-07-16 23:29:08
title: Javascript의 Array 함수들
updated:
---

일을 하던 도중 javascript의 배열 관련 함수를 자주 봤었다.
정확한 기능을 모르는 것도 있어서 구글링을 하곤 했는데 이게 반복되니 한 번 정리를 해야겠다 싶었다.

## 1. Array.length

기본이라고 말하기도 민망한 property지만 mdn을 보면서 새로운 사실을 알게 되어 정리하려고 한다.

{% blockquote MDN https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array#length_%EC%99%80_%EC%88%AB%EC%9E%90%ED%98%95_%EC%86%8D%EC%84%B1%EC%9D%98_%EA%B4%80%EA%B3%84 length 와 숫자형 속성의 관계 %}
    JavaScript 배열의 속성을 설정할 때 그 속성이 유효한 배열 인덱스이고 그 인덱스가 현재 배열의 경계(bounds)를 넘어간다면, 엔진은 배열의 length 속성을 그에 맞춰 업데이트 합니다:

    하지만, length 속성을 감소시키면 요소(element)를 지웁니다.
{% endblockquote %}

첫 번째 문장은 너무나 당연한 이야기다. 하지만 아래의 문장은 처음 알게 된 내용이었다.
즉, 아래와 같은 코드가 가능하다는 이야기

{% codeblock Array.length lang:JavaScript  %}
    let ary = [1, 2, 3]; // ary.length === 3
    ary.length = 1;
    console.log(ary);
    // [1]
{% endcodeblock %}

만약, length를 줄인 뒤에 다시 늘린다면 어떻게 될까?

{% codeblock Array.length lang:JavaScript  %}
    let ary = [1, 2, 3]; // ary.length === 3
    ary.length = 1;
    console.log(ary);
    // [1]
    ary.length = 3;
    // [1, undefined, undefined]
{% endcodeblock %}

역시나 undefined로 채워진다.

## 2. Array.from()

{% codeblock Array.from() lang:JavaScript  %}
    console.log(Array.from('foo'));
    // expected output: Array ["f", "o", "o"]

    console.log(Array.from([1, 2, 3], x => x + x));
    // expected output: Array [2, 4, 6]

    console.log(Array.from({ a: '1', b: '2' }));
    // [] : object는 빈 배열로 리턴된다.

    let obj = { a: '1', b: '2' };
    Array.from(Object.keys(obj));
    // ["a", "b"]
    Array.from(Object.values(obj));
    // ["1", "2"]
{% endcodeblock %}

## 3. Array.of()

{% codeblock Array.of() lang:JavaScript  %}
    Array.of(7);       // [7] 
    Array.of(1, 2, 3); // [1, 2, 3]
    Array.of(undefined); // [undefined]

    Array(7);          // [ , , , , , , ]
    Array(1, 2, 3);    // [1, 2, 3]
{% endcodeblock %}

## 4. Array.prototype.concat()

기존배열을 변경하지 않습니다. 
추가된 새로운 배열을 반환합니다.
중첩 배열 내부로 재귀하지 않습니다.

{% codeblock Array.prototype.concat() lang:JavaScript  %}
    const alpha = ['a', 'b', 'c'];
    const numeric = [1, 2, 3];

    alpha.concat(numeric);
    // 결과: ['a', 'b', 'c', 1, 2, 3]

    let b = [1, 2, 3];
    let c = [4, 5, [6, 7]];
    b.concat(c);
    // [1, 2, 3, 4, 5, [6, 7]]
{% endcodeblock %}