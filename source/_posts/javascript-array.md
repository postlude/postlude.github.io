---
categories:
  - JavaScript
tags:
  - javascript
date: 2019-07-16 23:29:08
title: Javascript의 Array 함수들
updated:
---


일을 하던 도중 JavaScript의 배열 관련 함수를 자주 봤었다.
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

## 15. Array.prototype.includes()

includes() 메서드는 배열이 특정 요소를 포함하고 있는지 판별합니다.

{% blockquote %}
    arr.includes(valueToFind[, fromIndex])
{% endblockquote %}

valueToFind
- 탐색할 요소. 문자나 문자열을 비교할 때, includes()는 대소문자를 구분합니다.

fromIndex (Optional)
- 이 배열에서 searchElement 검색을 시작할 위치입니다. 음의 값은 array.length + fromIndex의 인덱스를 asc로 검색합니다. 기본값은 0입니다.

{% codeblock Array.prototype.includes() lang:JavaScript  %}
    // array length is 3
    // fromIndex is -1
    // computed index is 3 + (-1) = 2

    var arr = ['a', 'b', 'c'];

    arr.includes('a', -1); // false
    arr.includes('a', -2); // false
    arr.includes('a', -3); // true

    var ary = [
        { a: 'a', b: 'b'},
        { a: 'aa', b: 'bb'}
    ];
    ary.includes({ a: 'a', b: 'b'}); // false. 객체는 확인 불가
{% endcodeblock %}

## 16. Array.prototype.indexOf()

indexOf() 메서드는 배열에서 지정된 요소를 찾을 수있는 첫 번째 인덱스를 반환하고 존재하지 않으면 -1을 반환합니다.

{% blockquote %}
    arr.indexOf(searchElement[, fromIndex])
{% endblockquote %}

searchElement
- 배열에서 찾을 요소입니다.

fromIndex (Optional)
- 검색을 시작할 색인입니다. 인덱스가 배열의 길이보다 크거나 같은 경우 -1이 반환되므로 배열이 검색되지 않습니다. 제공된 색인 값이 음수이면 배열 끝에서부터의 오프셋 값으로 사용됩니다.
- 제공된 색인이 음수이면 배열은 여전히 앞에서 뒤로 검색됩니다. 계산 된 인덱스가 0보다 작 으면 전체 배열이 검색됩니다. 기본값 : 0 (전체 배열 검색).

{% codeblock Array.prototype.indexOf() lang:JavaScript  %}
    var array = [2, 9, 9];
    array.indexOf(2);     // 0
    array.indexOf(7);     // -1
    array.indexOf(9, 2);  // 2
    array.indexOf(2, -1); // -1
    array.indexOf(2, -3); // 0

    var ary = [
        { a: 'a', b: 'b'},
        { a: 'aa', b: 'bb'}
    ];
    ary.indexOf({ a: 'a', b: 'b'}); // -1. 객체는 확인 불가
{% endcodeblock %}

## 17. Array.prototype.join()

join() 메서드는 배열의 모든 요소를 연결해 하나의 문자열로 만듭니다.

{% codeblock Array.prototype.join() lang:JavaScript  %}
    var elements = ['Fire', 'Air', 'Water'];

    console.log(elements.join());
    // expected output: "Fire,Air,Water"

    console.log(elements.join(''));
    // expected output: "FireAirWater"

    console.log(elements.join('-'));
    // expected output: "Fire-Air-Water"
{% endcodeblock %}

## 18. Array.prototype.keys()

keys() 메서드는 배열의 각 인덱스를 키 값으로 가지는 새로운 Array Iterator 객체를 반환합니다.

{% codeblock Array.prototype.keys() lang:JavaScript  %}
    var array1 = ['a', 'b', 'c'];
    var iterator = array1.keys(); 
    
    for (let key of iterator) {
    console.log(key); // expected output: 0 1 2
    }
{% endcodeblock %}

## 19. Array.prototype.keys()

keys() 메서드는 배열의 각 인덱스를 키 값으로 가지는 새로운 Array Iterator 객체를 반환합니다.

{% codeblock Array.prototype.keys() lang:JavaScript  %}
    var array1 = ['a', 'b', 'c'];
    var iterator = array1.keys(); 
    
    for (let key of iterator) {
        console.log(key); // expected output: 0 1 2
    }

    var arr = ['a', , 'c'];
    var sparseKeys = Object.keys(arr);
    var denseKeys = [...arr.keys()];
    console.log(sparseKeys); // ['0', '2']
    console.log(denseKeys);  // [0, 1, 2] 빈 값을 무시하지 않음
{% endcodeblock %}

## 20. Array.prototype.lastIndexOf()

lastIndexOf() 메서드는 지정된 요소가 배열에서 발견 될 수있는 마지막 인덱스를 반환하고, 존재하지 않으면 -1을 반환합니다. 배열은 fromIndex에서 시작하여 뒤로 검색됩니다.

## 21. Array.prototype.map()

map() 메서드는 배열 내의 모든 요소 각각에 대하여 주어진 함수를 호출한 결과를 모아 새로운 배열을 반환합니다.
map이 처리할 요소의 범위는 첫 callback을 호출하기 전에 정해집니다. map이 시작한 이후 배열에 추가되는 요소들은 callback을 호출하지 않습니다.

{% codeblock Array.prototype.map() lang:JavaScript  %}
    var array1 = [1, 4, 9, 16];

    // pass a function to map
    const map1 = array1.map(x => x * 2);

    console.log(map1); // expected output: Array [2, 8, 18, 32]
    console.log(array1); // Array [1, 4, 9, 16] 원본 유지

    // 아래 라인을 보시면...
    ['1', '2', '3'].map(parseInt);
    // 결과를 [1, 2, 3] 으로 기대할 수 있습니다.
    // 그러나 실제 결과는 [1, NaN, NaN] 입니다.

    // parseInt 함수는 보통 하나의 인자만 사용하지만, 두 개를 받을 수 있습니다.
    // 첫 번째 인자는 변환하고자 하는 표현이고 두 번째는 숫자로 변환할 때 사용할 진법입니다.
    // Array.prototype.map은 콜백에 세 가지 인자를 전달합니다.
    // 배열의 값, 값의 인덱스, 그리고 배열
    // 세 번째 인자는 parseInt가 무시하지만 두 번째 인자는 아닙니다.
{% endcodeblock %}

## 22. Array.prototype.pop()

pop() 메서드는 배열에서 마지막 요소를 제거하고 그 요소를 반환합니다.

## 23. Array.prototype.push()

push() 메서드는 배열의 끝에 하나 이상의 요소를 추가하고, 배열의 새로운 길이를 반환합니다.

## 24. Array.prototype.reduce()

reduce() 메서드는 배열의 각 요소에 대해 주어진 리듀서(reducer) 함수를 실행하고, 하나의 결과값을 반환합니다.

{% codeblock Array.prototype.reduce() lang:JavaScript  %}
    [0, 1, 2, 3, 4].reduce(function(accumulator, currentValue, currentIndex, array) {
        return accumulator + currentValue;
    });
{% endcodeblock %}

| callback | accumulator | currentValue | currentIndex | array | 반환 값 |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 1번째 호출 | 0 | 1 | 1 | [0, 1, 2, 3, 4] | 1 |
| 2번째 호출 | 1 | 2 | 2 | [0, 1, 2, 3, 4] | 3 |
| 3번째 호출 | 3 | 3 | 3 | [0, 1, 2, 3, 4] | 6 |
| 4번째 호출 | 6 | 4 | 4 | [0, 1, 2, 3, 4] | 10 |

{% codeblock Array.prototype.reduce() lang:JavaScript  %}
    [0, 1, 2, 3, 4].reduce(function(accumulator, currentValue, currentIndex, array) {
        return accumulator + currentValue;
    }, 10);
{% endcodeblock %}

| callback | accumulator | currentValue | currentIndex | array | 반환 값 |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 1번째 호출 | 10 | 0 | 0 | [0, 1, 2, 3, 4] | 10 |
| 2번째 호출 | 10 | 1 | 1 | [0, 1, 2, 3, 4] | 11 |
| 3번째 호출 | 11 | 2 | 2 | [0, 1, 2, 3, 4] | 13 |
| 4번째 호출 | 13 | 3 | 3 | [0, 1, 2, 3, 4] | 16 |
| 5번째 호출 | 16 | 4 | 4 | [0, 1, 2, 3, 4] | 20 |

## 25. Array.prototype.reduceRight()

reduceRight() 메서드는 누적기에 대해 함수를 적용하고 배열의 각 값 (오른쪽에서 왼쪽으로)은 값을 단일 값으로 줄여야합니다.
reduce와 동일하되 오른쪽에서 왼쪽으로 reducer 함수 호출.

## 26. Array.prototype.reverse()

배열의 순서를 반전합니다.

## 27. Array.prototype.shift()

shift() 메서드는 배열에서 첫 번째 요소를 제거하고, 제거된 요소를 반환합니다. 이 메서드는 배열의 길이를 변하게 합니다.

## 28. Array.prototype.slice()

slice() 메서드는 어떤 배열의 begin부터 end까지(end 미포함)에 대한 얕은 복사본을 새로운 배열 객체로 반환합니다. 원본 배열은 수정되지 않습니다.

{% codeblock Array.prototype.slice() lang:JavaScript  %}
    var animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

    const ary1 = animals.slice(2);
    const ary2 = animals.slice(2, 4)

    console.log(animals);
    console.log(ary1);
    console.log(ary2);

    animals[0] = 'aaaaa';
    console.log(animals);
    console.log(ary1);

    // Array ["ant", "bison", "camel", "duck", "elephant"]
    // Array ["camel", "duck", "elephant"]
    // Array ["camel", "duck"]
    // Array ["aaaaa", "bison", "camel", "duck", "elephant"]
    // Array ["camel", "duck", "elephant"]
{% endcodeblock %}

## 29. Array.prototype.some()

some() 메서드는 배열 안의 어떤 요소라도 주어진 판별 함수를 통과하는지 테스트합니다.
빈 배열에서 호출하면 무조건 false를 반환합니다.

{% codeblock Array.prototype.some() lang:JavaScript  %}
    var array = [1, 2, 3, 4, 5];
    var array2 = [1, 2, 3, 3, 5];

    var even = function(element) {
        // checks whether an element is even
        return element % 2 === 0;
    };

    console.log(array.some(even));
    // expected output: true

    console.log(array2.some(even));
    // expected output: true
{% endcodeblock %}

## 30. Array.prototype.sort()

sort() 메서드는 배열의 요소를 적절한 위치에 정렬한 후 그 배열을 반환합니다. 정렬은 stable sort가 아닐 수 있습니다.
기본 정렬 순서는 문자열의 유니코드 코드 포인트를 따릅니다.(오름차순)

{% blockquote %}
    stable sort : 졍렬시 같은 값에 대하여 원본 순서가 유지되는 정렬
    unstable sort : 졍렬시 같은 값에 대하여 원본 순서가 유지되지 않는 정렬
{% endblockquote %}

## 31. Array.prototype.splice()

splice() 메서드는 배열의 기존 요소를 삭제 또는 교체하거나 새 요소를 추가하여 배열의 내용을 변경합니다.
만약 제거할 요소의 수와 추가할 요소의 수가 다른 경우 배열의 길이는 달라집니다.

{% blockquote %}
    array.splice(start[, deleteCount[, item1[, item2[, ...]]]])
{% endblockquote %}

start
- 배열의 변경을 시작할 인덱스입니다. 배열의 길이보다 큰 값이라면 실제 시작 인덱스는 배열의 길이로 설정됩니다.
음수인 경우 배열의 끝에서부터 요소를 세어나갑니다(원점 -1, 즉 -n이면 요소 끝의 n번째 요소를 가리키며 array.length - n번째 인덱스와 같음).
값의 절대값이 배열의 길이 보다 큰 경우 0으로 설정됩니다.

deleteCount (Optional)
- 배열에서 제거할 요소의 수입니다.
deleteCount를 생략하거나 값이 array.length - start보다 크면 start부터의 모든 요소를 제거합니다.
deleteCount가 0 이하라면 어떤 요소도 제거하지 않습니다. 이 때는 최소한 하나의 새로운 요소를 지정해야 합니다.

item1, item2, ... (Optional)
- 배열에 추가할 요소입니다. 아무 요소도 지정하지 않으면 splice()는 요소를 제거하기만 합니다.

{% codeblock Array.prototype.splice() lang:JavaScript  %}
    var months = ['Jan', 'March', 'April', 'June'];
    months.splice(1, 0, 'Feb');
    // inserts at index 1
    console.log(months);
    // expected output: Array ['Jan', 'Feb', 'March', 'April', 'June']

    months.splice(4, 1, 'May');
    // replaces 1 element at index 4
    console.log(months);
    // expected output: Array ['Jan', 'Feb', 'March', 'April', 'May']

    months.splice(1, 2, 'May');
    console.log(months);
    // expected output: Array ["Jan", "May", "April", "May"]
{% endcodeblock %}

## 32. Array.prototype.toLocaleString()

toLocaleString() 메서드는 배열의 요소를 나타내는 문자열을 반환합니다.
요소는 toLocaleString 메서드를 사용하여 문자열로 변환되고 이 문자열은 locale 고유 문자열(가령 쉼표 “,”)에 의해 분리됩니다.

{% codeblock Array.prototype.toLocaleString() lang:JavaScript  %}
    var array1 = [1, 'a', new Date('21 Dec 1997 14:12:00 UTC')];
    var localeString = array1.toLocaleString('en', {timeZone: "UTC"});

    console.log(localeString);
    // expected output: "1,a,12/21/1997, 2:12:00 PM",
    // This assumes "en" locale and UTC timezone - your results may vary
{% endcodeblock %}

## 33. Array.prototype.toString()

toString() 메서드는 지정된 배열 및 그 요소를 나타내는 문자열을 반환합니다.

{% codeblock Array.prototype.splice() lang:JavaScript  %}
    var array1 = [1, 2, 'a', '1a'];

    console.log(array1.toString());
    // expected output: "1,2,a,1a"
{% endcodeblock %}

## 34. Array.prototype.unshift()

unshift() 메서드는 새로운 요소를 배열의 맨 앞쪽에 추가하고, 새로운 길이를 반환합니다.

{% codeblock Array.prototype.unshift() lang:JavaScript  %}
    var array1 = [1, 2, 3];

    console.log(array1.unshift(4, 5));
    // expected output: 5

    console.log(array1);
    // expected output: Array [4, 5, 1, 2, 3]


    var arr = [1, 2];

    arr.unshift(0); // result of call is 3, the new array length
    // arr is [0, 1, 2]

    arr.unshift(-2, -1); // = 5
    // arr is [-2, -1, 0, 1, 2]

    arr.unshift([-3]);
    // arr is [[-3], -2, -1, 0, 1, 2]
{% endcodeblock %}

## 35. Array.prototype.values()

values() 메서드는 배열의 각 인덱스에 대한 값을 가지는 새로운 Array Iterator 객체를 반환합니다.

{% codeblock Array.prototype.values() lang:JavaScript  %}
    const array1 = ['a', 'b', 'c'];
    const iterator = array1.values();

    for (const value of iterator) {
        console.log(value); // expected output: "a" "b" "c"
    }


    var arr = ['w', 'y', 'k', 'o', 'p'];
    var eArr = arr.values();
    console.log(eArr.next().value); // w
    console.log(eArr.next().value); // y
    console.log(eArr.next().value); // k
    console.log(eArr.next().value); // o
    console.log(eArr.next().value); // p
{% endcodeblock %}

