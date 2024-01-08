---
categories:
  - JavaScript
tags:
  - javascript
date: 2019-02-04 16:41:50
title: JavaScript의 Property Descriptor
updated:
---

JavaScript의 Property Descriptor에 대해 정리해보겠습니다.
Property Descriptor란 말 그대로 Property를 설명해주는 객체를 말합니다.
Property Descriptor는 `Object.getOwnPropertyDescriptor()` 함수를 통해 가져올 수 있습니다.

{% codeblock lang:JavaScript %}
    var person = {
        name: 'postlude'
    };
    var nameDescriptor = Object.getOwnPropertyDescriptor(person, 'name');

    console.log(nameDescriptor);
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    {
        value: 'postlude',
        writable: true,
        enumerable: true,
        configurable: true
    }
{% endcodeblock %}

Property Descriptor는 2가지 타입으로 나뉩니다. data descriptor와 accessor descriptor입니다.

{% blockquote MDN https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty Object.defineProperty() %}
    A data descriptor is a property that has a value, which may or may not be writable. An accessor descriptor is a property described by a getter-setter pair of functions. A descriptor must be one of these two flavors; it cannot be both.
{% endblockquote %}

위의 코드에서 여러 가지로 테스트해보겠습니다.

# 1. Data Descriptor
## 1.1 Default Value
Data Descriptor란 value를 가진 property를 말합니다. 위의 예시에서 `nameDescriptor`가 Data Descriptor가 그 예시입니다.
property를 추가할 때 `Object.defineProperty()` 함수를 이용하면 property descriptor를 바꿔서 추가할 수 있습니다.

{% codeblock lang:JavaScript %}
    Object.defineProperty(person, 'age', {
        value: 10
    });
    var ageDescriptor = Object.getOwnPropertyDescriptor(person, 'age');

    console.log(ageDescriptor);
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    {
        value: 10,
        writable: false,
        enumerable: false,
        configurable: false
    }
{% endcodeblock %}

`Object.defineProperty()` 함수를 이용하여 property를 추가할 때 값을 넣지 않으면 리터럴 형식으로 넣는 것과는 다르게 default가 false라는 것을 알 수 있습니다.

## 1.2 enumerable

위와 같은 상태에서 person을 출력해보겠습니다.
{% codeblock lang:JavaScript %}
    console.log(person);
    console.log(person.age);
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    { name: 'postlude' }
    10
{% endcodeblock %}

person 을 출력해도 age는 보이지 않습니다. 하지만 `Object.defineProperty()` 를 이용해 age 라는 property를 추가했기 때문에 명시적으로 호출하면 값을 출력할 수 있습니다.
즉, enumerable 은 해당 property가 보일지 말지를 결정하는 속성입니다.

## 1.3 writable

이름에서 바로 느껴지듯이 writable은 해당 속성이 = 를 이용해 새로운 값을 할당할 수 있는가에 대한 속성입니다.
(enumerable은 true로 변경하고 진행했습니다.)

{% codeblock lang:JavaScript %}
    Object.defineProperty(person, 'age', {
        value: 10,
        enumerable: true,
    });
    var ageDescriptor = Object.getOwnPropertyDescriptor(person, 'age');

    console.log(ageDescriptor);
    console.log(person);
    person.age = 20;
    console.log(person.age);
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    {
        value: 10,
        writable: false,
        enumerable: true,
        configurable: false
    }
    { name: 'postlude', age: 10 }
    10
{% endcodeblock %}

writable이 false이기 때문에 person.age 를 새로운 값으로 바꿀 수 없습니다.

## 1.4 configurable

configurable은 다음과 같이 설명되어 있습니다.

{% blockquote MDN https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty Object.defineProperty() %}
    true if and only if the type of this property descriptor may be changed and if the property may be deleted from the corresponding object.
{% endblockquote %}

property descriptor를 변경할 수 있는가에 대한 내용입니다.

{% codeblock lang:JavaScript %}
    Object.defineProperty(person, 'age', {
        value: 10,
    });
    Object.defineProperty(person, 'age', {
        value: 20,
        enumerable: true,
        writable: true
    });
{% endcodeblock %}

위와 같이 age를 추가한 후에 다시 age를 추가하려고 시도하면

{% codeblock lang:JavaScript %}
    // result
    Object.defineProperty(person, 'age', {
       ^
    TypeError: Cannot redefine property: age
{% endcodeblock %}

에러가 발생합니다.
만약 configurable 이 true라면

{% codeblock lang:JavaScript %}
    Object.defineProperty(person, 'age', {
        value: 10,
    });
    Object.defineProperty(person, 'age', {
        value: 20,
        enumerable: true,
        writable: true
    });

    console.log(person);
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    { name: 'postlude', age: 20 }
{% endcodeblock %}

정상적으로 동작하게 됩니다.

# 2. Accessor Descriptor

Accessor Descriptor는 getter와 setter로 표현되는 descriptor입니다.
Accessor Descriptor도 동일하게 configurable과 enumerable을 가지고 있으며 data descriptor와 동일하게 동작합니다.

{% codeblock lang:JavaScript %}
    Object.defineProperty(person, 'job', {
        enumerable: true,
        get: function() {
            console.log('job getter');
        },
        set: function() {
            console.log('job setter');
        }
    });
    var jobDescriptor = Object.getOwnPropertyDescriptor(person, 'job');

    console.log(jobDescriptor);
    person.job;
    person.job = 'developer';
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    {
        get: [Function: get],
        set: [Function: set],
        enumerable: true,
        configurable: false
    }
    job getter
    job setter
{% endcodeblock %}

get으로 지정된 함수는 property를 읽을 때, set으로 지정된 함수는 property를 세팅할 때 자동적으로 호출되는 함수입니다.

# 3. Private 변수

위의 내용을 바탕으로 외부에서 접근 불가능한 private 변수를 JavaScript에서도 비슷하게 흉내낼 수 있습니다.

{% codeblock lang:JavaScript %}
    Object.defineProperty(person, '_job', {
        value: 'student',
        writable: true
    })
    Object.defineProperty(person, 'job', {
        enumerable: true,
        get: function() {
            return this._job;
        },
        set: function(value) {
            this._job = value;
        }
    });
    var jobDescriptor = Object.getOwnPropertyDescriptor(person, 'job');

    console.log(jobDescriptor);
    console.log(person.job);
    person.job = 'developer';
    console.log(person.job);
{% endcodeblock %}
{% codeblock lang:JavaScript %}
    // result
    {
        get: [Function: get],
        set: [Function: set],
        enumerable: true,
        configurable: false
    }
    student
    developer
{% endcodeblock %}

실제 데이터는 _job에 들어가지마 enumerable이 false 이기 때문에 외부에서 확인이 불가능합니다.
호출할 수 있는 property는 job이고, job을 통해 _job의 값을 읽거나 쓸 수 있게 됩니다.
물론 `person._job = 'test';` 라는 식으로 코드를 작성하면 값을 변경할 수 있게 됩니다. 완벽한 private 변수는 아닌 셈이지요.

> 사실 private 변수를 이용하기 위해선 클로저로도 가능합니다. 다른 포스트로 알아보겠습니다.
