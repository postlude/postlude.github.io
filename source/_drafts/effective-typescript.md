---
categories:
  - null
tags:
  - null
date: 2024-01-14 22:42:31
title: 이펙티브 타입스크립트
updated:
---

# 1장 타입스크립트 알아보기

{% asset_img 1.png %}

- 타입스크립트는 자바스크립트의 상위집합
= 모든 js 프로그램은 ts 프로그램
(ts는 일반적으로는 유효한 js 프로그램이 아님)
- 타입 오류가 있어도 컴파일은 가능

# 2장 타입스크립트의 타입 시스템

- 타입 : 값의 집합
- A는 B를 상속
= A는 B에 할당 가능
= A는 B의 서브타입
= A는 B의 부분 집합
- 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.

- unknown
{% codeblock unknown lang:TypeScript %}
	interface Person { name: string; }
    const body = document.body;
    
    // HTMLElement -> Person 형식이 다른 형식과 충분히 겹치치 않기 때문에 오류
    const el = body as Person;
    
    // 정상 but unknown은 항상 위험을 내포하고 있음
    const el = body as unknown as Person;
{% endcodeblock %}

- 매개변수나 반환 값에 타입을 명시하기보다 함수 표현식 전체에 타입 구문을 적용시키는게 좋음
{% codeblock lang:TypeScript %}
	// bad
	function add(a: number, b: number) { return a + b; }
	function sub(a: number, b: number) { return a - b; }
	function mul(a: number, b: number) { return a * b; }
	function div(a: number, b: number) { return a / b; }

	// good
	type BinaryFn = (a: number, b: number) => number;
	const add: BinaryFn = (a, b) => a + b;
	const sub: BinaryFn = (a, b) => a - b;
	const mul: BinaryFn = (a, b) => a * b;
	const div: BinaryFn = (a, b) => a / b;
{% endcodeblock %}

- 다른 함수의 시그니처를 참조하려면 `typeof fn` 을 사용하면 됨
{% codeblock typeof function lang:TypeScript %}
	const checkedFetch: typeof fetch = async (input, init) => { ... }
{% endcodeblock %}

- DRY(Don’t Repeat Yourself) 원칙을 타입에도 최대한 적용할 것
- readonly
	- `readonly number[]`
		- element 읽기만 가능
		- length 변경 불가(읽기 가능)
		- pop, shift 등의 함수 호출 불가
	- 함수 내에서 매개변수를 변경하지 않으면 매개변수에 readonly를 붙이자
		- 오류 발견이 쉬워짐
		- 파라미터의 경우 배열과 같은 형태에서만 붙일 수 있음
		{% asset_img 2.png %}
	- 얕게(shallow) 동작함
    	- 배열 element가 객체면 객체의 프로퍼티는 변경 가능
	- index signature 에도 적용 가능
	{% codeblock index signature lang:TypeScript %}
		class A {
			readonly [key: string]: number
		}

		// 가능
		const a: A {
			aa: 1,
			bb: 2
		};

		// 불가능
		a['aa'] = 1;
		a.bb = 2;
	{% endcodeblock %}

# 3장 타입 추론

- 타입 추론이 가능한 경우 명시적 타입 구문은 불필요
- 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려
    - 내부 구현의 오류가 사용자 코드 위치(함수 호출 부분 등)에 나타나는 것을 방지
- 상수를 이용해 변수 초기화시 타입을 명시하지 않으면 타입 체크는 타입을 결정해야 함
    - 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다
    - 타입 넓히기(widening)
- const 단언문
    - 변수 선언에 쓰이는 let, const와 다름
    - 값 뒤에 `as const` 를 사용하면 ts는 최대한 좁은 타입으로 추론
	{% codeblock as const lang:TypeScript %}
		const v1 = { x: 1, y: 2 }; // 타입 : { x: number; y: number; }
		const v2 = { x: 1 as const, y: 2 }; // 타입 : { x: 1; y: number; }
		const v3 = { x: 1, y: 2 } as const; // 타입 : { readonly x: 1; readonly y: 2; }
	{% endcodeblock %}
	- type narrowing -> destructuring 을 사용하는게 좋음

# 4장 타입 설계

- 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 오류를 유발하게 된다.
    - 유효한 상태만 표현하는 타입을 지향해야 함
	{% codeblock lang:TypeScript %}
		// bad
		interface State {
		pageText: string;
		isLoading: boolean;
		error?: string;
		}

		// good
		interface RequestPending { state: 'pending'; }
		interface RequestError { state: 'error'; error: string; }
		interface RequestSuccess { state:: 'ok'; pageText: string; }

		type RequestState = RequestPending | RequestError | RequestSuccess;
	{% endcodeblock %}
	- 함수 매개변수 타입의 범위는 넓어도 되지만, 리턴 타입은 좁은게 좋음
	- 주석과 변수명에 타입 정보를 적지 말 것
		- 단위 정보는 포함해도 됨(ex. timeMs)
	- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 발생할 수 있음 - 주의
	- `string` 타입보다는 가능한한 구체적인 타입을 사용(ex. `type A = 'a' | 'b'`)

# 5장 any 다루기

- any의 사용 범위는 최소화해야 된다.
{% codeblock any lang:TypeScript %}
	// bad
	function f1() {
		const x: any = returningFoo();
		processBar(x);
	}

	// good
	function f2() {
		const x = returningFoo();
		processBar(x as any);
	}
{% endcodeblock %}
- 함수 반환 타입이 any인 경우 타입 안정성이 나빠짐(any 리턴하지 말 것)
- any를 사용할 때는 정말로 모든 값이 허용되어야 하는지 검토
- 타입 단언문 : 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안에서
- any의 진화 - but 명시적 타입 구문을 사용하는게 좋음
{% codeblock any evolve lang:TypeScript %}
	function range(start: number, limit: number) {
		const out = []; // any[]
		for (let i = start; i < limit; i++) {
			out.push(i); // any[]
		}
		return out; // number[]
	}
{% endcodeblock %}
- 