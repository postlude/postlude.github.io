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

## 5. Array.prototype.copyWithin()

배열의 일부를 얕게 복사한 뒤, 동일한 배열의 다른 위치에 덮어쓰고 그 배열을 반환합니다. 이 때, 크기(배열의 길이)를 수정하지 않고 반환합니다.

{% blockquote %}
    arr.copyWithin(target[, start[, end]])
{% endblockquote %}

target
- 복사한 시퀀스(값)를 넣을 위치를 가리키는 0 기반 인덱스. 음수를 지정하면 인덱스를 배열의 끝에서부터 계산합니다.
target이 arr.length보다 크거나 같으면 아무것도 복사하지 않습니다. target이 start 이후라면 복사한 시퀀스를 arr.length에 맞춰 자릅니다.

start (Optional)
- 복사를 시작할 위치를 가리키는 0 기반 인덱스. 음수를 지정하면 인덱스를 배열의 끝에서부터 계산합니다.
기본값은 0으로, start를 지정하지 않으면 배열의 처음부터 복사합니다.

end (Optional)
- 복사를 끝낼 위치를 가리키는 0 기반 인덱스. copyWithin은 end 인덱스 이전까지 복사하므로 end 인덱스가 가리키는 요소는 제외합니다. 음수를 지정하면 인덱스를 배열의 끝에서부터 계산합니다.
기본값은 arr.length로, end를 지정하지 않으면 배열의 끝까지 복사합니다.

{% codeblock Array.prototype.copyWithin() lang:JavaScript  %}
    [1, 2, 3, 4, 5].copyWithin(-2);
    // [1, 2, 3, 1, 2]

    [1, 2, 3, 4, 5].copyWithin(0, 3);
    // [4, 5, 3, 4, 5]

    [1, 2, 3, 4, 5].copyWithin(0, 3, 4);
    // [4, 2, 3, 4, 5]

    [1, 2, 3, 4, 5].copyWithin(-2, -3, -1);
    // [1, 2, 3, 3, 4]
{% endcodeblock %}

## 6. Array.prototype.entries()

배열의 각 인덱스에 대한 키/값 쌍을 가지는 새로운 Array Iterator 객체를 반환합니다.

{% codeblock Array.prototype.entries() lang:JavaScript  %}
    var array1 = ['a', 'b', 'c'];
    var iterator1 = array1.entries();

    console.log(iterator1.next().value);
    // expected output: Array [0, "a"]

    console.log(iterator1.next().value);
    // expected output: Array [1, "b"]

    var a = ['a', 'b', 'c'];
    var iterator = a.entries();

    for (let e of iterator) {
        console.log(e);
    }
    // [0, 'a']
    // [1, 'b']
    // [2, 'c']
{% endcodeblock %}

## 7. Array.prototype.every()

배열 안의 모든 요소가 주어진 판별 함수를 통과하는지 테스트합니다.
every는 호출한 배열을 변형하지 않습니다.

{% codeblock Array.prototype.every() lang:JavaScript  %}
    function isBelowThreshold(currentValue, index, array) {
        console.log(index);
        return currentValue < 40;
    }

    var array1 = [1, 30, 45, 29, 10, 13];

    console.log(array1.every(isBelowThreshold));
    // 0
    // 1
    // false
{% endcodeblock %}

## 8. Array.prototype.fill()

배열의 시작 인덱스부터 끝 인덱스의 이전까지 정적인 값 하나로 채웁니다.

{% codeblock Array.prototype.fill() lang:JavaScript  %}
    var array1 = [1, 2, 3, 4];

    // fill with 0 from position 2 until position 4
    console.log(array1.fill(0, 2, 4));
    // expected output: [1, 2, 0, 0]

    // fill with 5 from position 1
    console.log(array1.fill(5, 1));
    // expected output: [1, 5, 5, 5]

    console.log(array1.fill(6));
    // expected output: [6, 6, 6, 6]
{% endcodeblock %}

## 9. Array.prototype.filter()

주어진 함수의 테스트를 통과하는 모든 요소를 모아 새로운 배열로 반환합니다.
원본 배열은 그대로 유지.

{% codeblock Array.prototype.filter() lang:JavaScript  %}
    var words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

    const result = words.filter(word => word.length > 6);

    console.log(words);
    console.log(result);
    // Array ["spray", "limit", "elite", "exuberant", "destruction", "present"]
    // Array ["exuberant", "destruction", "present"]
{% endcodeblock %}

## 10. Array.prototype.find()

주어진 판별 함수를 만족하는 첫 번째 요소의 값을 반환합니다. 그런 요소가 없다면 undefined를 반환합니다.

{% codeblock Array.prototype.find() lang:JavaScript  %}
    var array1 = [5, 12, 8, 130, 44];

    var found = array1.find(function(element) {
        return element > 10;
    });

    console.log(found);
    // expected output: 12

    const inventory = [
        {name: 'apples', quantity: 2},
        {name: 'bananas', quantity: 0},
        {name: 'cherries', quantity: 5}
    ];

    const result = inventory.find(fruit => fruit.name === 'cherries');

    console.log(result) // { name: 'cherries', quantity: 5 }
{% endcodeblock %}

## 11. Array.prototype.findIndex()

주어진 판별 함수를 만족하는 배열의 첫 번째 요소에 대한 인덱스를 반환합니다. 만족하는 요소가 없으면 -1을 반환합니다.

{% codeblock Array.prototype.findIndex() lang:JavaScript  %}
    var array1 = [5, 12, 8, 130, 44];

    function isLargeNumber(element) {
        return element > 13;
    }

    console.log(array1.findIndex(isLargeNumber));
    // expected output: 3

{% endcodeblock %}

## 12. Array.prototype.flat()

모든 하위 배열 엘리먼트를 지정된 깊이까지 재귀적으로 이어붙여 새로운 배열을 생성합니다.

{% codeblock Array.prototype.flat() lang:JavaScript  %}
    var arr1 = [1, 2, [3, 4]];
    arr1.flat(); 
    // [1, 2, 3, 4]

    var arr2 = [1, 2, [3, 4, [5, 6]]];
    arr2.flat();
    // [1, 2, 3, 4, [5, 6]]

    var arr3 = [1, 2, [3, 4, [5, 6]]];
    arr3.flat(2);
    // [1, 2, 3, 4, 5, 6]

    var arr4 = [1, 2, , 4, 5];
    arr4.flat();
    // [1, 2, 4, 5]
{% endcodeblock %}

## 13. Array.prototype.flatMap()

매핑함수를 사용해 각 엘리먼트에 대해 map 수행 후, 결과를 새로운 배열로 평평화합니다.

{% codeblock Array.prototype.flatMap() lang:JavaScript  %}
    let arr1 = [1, 2, 3, 4];

    arr1.map(x => [x * 2]); 
    // [[2], [4], [6], [8]]

    arr1.flatMap(x => [x * 2]);
    // [2, 4, 6, 8]

    // 한 레벨만 평평화됨
    arr1.flatMap(x => [[x * 2]]);
    // [[2], [4], [6], [8]]

    let arr2 = ["it's Sunny in", "", "California"];

    arr2.map(x=>x.split(" "));
    // [["it's","Sunny","in"],[""],["California"]]

    arr2.flatMap(x => x.split(" "));
    // ["it's","Sunny","in","California"]
{% endcodeblock %}

## 14. Array.prototype.forEach()

주어진 함수를 배열 요소 각각에 대해 실행합니다.
forEach()는 배열을 변형하지 않습니다. 그러나 callback이 변형할 수는 있습니다.
예외를 던지지 않고는 forEach()를 중간에 멈출 수 없습니다.



