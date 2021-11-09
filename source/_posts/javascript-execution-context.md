---
categories:
  - JavaScript
tags:
  - javascript
date: 2021-11-07 16:54:31
title: JavaScript의 실행 컨텍스트(Execution Context)
updated:
---

## 0. 배경

이전 이야기를 조금 해보겠습니다.

회사에 들어오고 얼마 지나지 않았을 때 js에 대해 공부를 해야겠다고 생각했었습니다.
업무를 하면서 뭔가 언어의 근본적인 부분에 대해 모르고 쓰고 있다는 생각이 들었기 때문입니다.

그래서 js관련 오프라인 강의를 들었습니다. 그 때 실행 컨텍스트라는 단어도 처음 들었습니다.

강의를 들을 때도 100% 이해가 된 건 아니었습니다. 강의가 끝난 후에 공부를 해야지라는 생각을 하고 시도를 했었는데 잘 이해가 안되더군요(...)

그래서 그 당시에 정리를 위한 draft를 만들었었는데 그걸 이때까지 publish를 하지 못하고 있다가 최근에 다시 공부하면서 조금씩 이해가 되어서 이렇게 정리를 해보려고 합니다.

{% blockquote %}
    완벽하게 이해하지 못한 부분도 있습니다.
{% endblockquote %}

{% blockquote %}
    언제나 그렇듯 잘못된 정보가 있으면 알려주시면 감사하겠습니다.
{% endblockquote %}

## 1. 실행 컨텍스트(Execution Context)란?

ECMAScript 2015 문서에 따르면 Excution Context(이하 EC)의 정의는 아래와 같습니다.

{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts Execution Contexts %}
    An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation. At any point in time, there is at most one execution context that is actually executing code. This is known as the running execution context. A stack is used to track execution contexts. The running execution context is always the top element of this stack. A new execution context is created whenever control is transferred from the executable code associated with the currently running execution context to executable code that is not associated with that execution context. The newly created execution context is pushed onto the stack and becomes the running execution context.
{% endblockquote %}

첫 문장부터 의문이 생겼습니다. **runtime evaluation**은 뭘까요? 구글링을 통해 찾은 의미는 다음과 같았습니다.

{% blockquote rosettacode.org https://rosettacode.org/wiki/Runtime_evaluation Runtime evaluation %}
    Demonstrate a language's ability for programs to execute code written in the language provided at runtime.
{% endblockquote %}

즉, 실행 평가(runtime evaluation)란 어떤 코드가 런타임에 어떤 기능을 하는지 판단하는 것입니다.

다시 EC로 돌아오면, EC의 역할은 이 실행 평가를 추적하는 것입니다.
마지막 두 문장을 보면 컨트롤이 실행 가능한 코드(executable code)로 옮겨가면 새로운 EC가 생성되고 stack에 push된다고 나와있습니다. 그리고 실행 중인 EC는 새로 생성된 EC가 된다고 합니다.

문서에 나와있는 소스 코드의 종류는 다음과 같습니다.

{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-types-of-source-code Types of Source Code %}
    Global code is source text that is treated as an ECMAScript Script. The global code of a particular Script does not include any source text that is parsed as part of a FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.

    Eval code is the source text supplied to the built-in eval function. More precisely, if the parameter to the built-in eval function is a String, it is treated as an ECMAScript Script. The eval code for a particular invocation of eval is the global code portion of that Script.

    Function code is source text that is parsed to supply the value of the [[ECMAScriptCode]] and [[FormalParameters]] internal slots (see 9.2) of an ECMAScript function object. The function code of a particular ECMAScript function does not include any source text that is parsed as the function code of a nested FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.

    Module code is source text that is code that is provided as a ModuleBody. It is the code that is directly evaluated when a module is initialized. The module code of a particular module does not include any source text that is parsed as part of a nested FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.
{% endblockquote %}

일단 `Global code`와 `Function code`에 집중해서 이해해보도록 하겠습니다.

## 2. Execution Stack

EC가 쌓이는 stack을 Execution Stack이라고 부릅니다. 사실 처음에 단어를 들었을 때는 이해가 가지 않았는데 좀 더 익숙한 단어로 바꾸면 **call stack**입니다.
({% link Is "Call stack" the same as "Execution context stack" in JavaScript? https://stackoverflow.com/questions/55140096/is-call-stack-the-same-as-execution-context-stack-in-javascript %})

아래와 같은 코드가 있다고 가정해보겠습니다.

{% codeblock lang:JavaScript %}
    let gb = 'global variable';

    function func1() {
        console.log('start func1');
        func2();
        console.log('end func1');
    }

    function func2() {
        console.log('func2');
    }

    func1();

    console.log('global execution context');
{% endcodeblock %}

맨 처음 글로벌 ec가 만들어지고 func1이 호출될 때 해당 함수의 ec가 만들어 집니다. func1 내부에서 func2를 호출할 때도 마찬가지로 func2의 ec가 만들어 집니다.

ec가 만들어지는 순서를 보면 아래와 같습니다.

{% asset_img 1.JPG %}

## 3. EC의 구성 요소

EC는 아래와 같은 구조로 되어 있습니다.

{% codeblock lang:JavaScript %}
    ExecutionContext: {
        LexicalEnvironment,
        VariableEnvironment
    }
{% endcodeblock %}

### 3.1 Lexical Environment

LE의 정의는 다음과 같습니다.

{% blockquote ECMAScript® 2015 Language Specification https://262.ecma-international.org/6.0/#sec-lexical-environments Lexical Environments %}
    A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code.
{% endblockquote %}

다시 말해 변수, 함수에 대한 식별자 바인딩 정보를 담고 있는 곳입니다. LE는 다시 EnvironmentRecord와 OuterLexicalEnvironment로 구성되어 있습니다.

{% codeblock lang:JavaScript %}
    LexicalEnvironment: {
        EnvironmentRecord,
        OuterLexicalEnvironment
    }
{% endcodeblock %}

EnvironmentRecord는 위에서 말한 LE의 식별자 바인딩 정보를 가지고 있습니다.

OuterLexicalEnvironment는 이름에서 상위 LE에 대한 정보를 가지고 있습니다. 함수 내에 존재하지 않지만 자신이 속한 상위 함수나 글로벌 영역에 있는 변수나 함수를 참조할 수 있는 것은 바로 OuterLexicalEnvironment가 있기 때문에 가능한 것입니다.

### 3.2 Variable Environment

ES6 에서 LexicalEnvironment와 VariableEnvironment의 차이점은 전자가 함수 선언과 변수(let, const)의 바인딩을 저장하고 후자는 변수(var) 만 저장하는 것이라고 합니다.

전체 구조로 보면 EC는 아래와 같은 모습입니다.
(사실 이것 외에도 다른 부분들이 더 있으나 제가 이해한 영역만 작성했습니다.)

{% codeblock lang:JavaScript %}
    ExecutionContext: {
        LexicalEnvironment: {
            EnvironmentRecord,
            OuterLexicalEnvironment
        },
        VariableEnvironment: {
            EnvironmentRecord,
            OuterLexicalEnvironment
        }
    }
{% endcodeblock %}

## 4. 예시

아래의 코드를 예시로 이해해보도록 하겠습니다.

{% codeblock lang:JavaScript %}
    let a = 20;
    const b = 30;
    var c;

    function multiply(e, f) {
        var g = 20;
        return e * f * g;
    }

    c = multiply(20, 30);
{% endcodeblock %}

### 4.1 Global EC 생성과 Hoisting

가장 먼저 global EC가 아래와 같이 만들어집니다.

{% codeblock lang:JavaScript %}
    GlobalExectionContext = {
        LexicalEnvironment: {
            EnvironmentRecord: {
                a: < uninitialized >,
                b: < uninitialized >,
                multiply: < func >
            }
            OuterLexicalEnvironment: null
        },
        VariableEnvironment: {
            EnvironmentRecord: {
                c: undefined
            }
            OuterLexicalEnvironment: null
        }
    }
{% endcodeblock %}

여기서 중요한 점은 a, b, c 변수의 차이점입니다.
let, const로 선언된 a, b 변수는 **초기화 되지 않은 상태**이며 var로 선언된 c는 **undefined**가 할당되어 있습니다.

이런 차이로 인해 var로 선언된 변수는 코드 상에서 선언된 위치보다 위에서 접근할 수 있고, let과 const로 선언된 변수는 선언 위치보다 먼저 접근할 경우 reference error가 발생하게 됩니다.
**Hoisting이라고 부르는 현상이 바로 이것입니다.**

### 4.2 코드 실행

이제 코드가 첫 줄부터 실행되고 a, b 변수에 값이 할당됩니다.

{% codeblock lang:JavaScript %}
    GlobalExecutionContext = {
        LexicalEnvironment: {
            EnvironmentRecord: {
                a: 20,
                b: 30,
                multiply: < func >
            }
            OuterLexicalEnvironment: null
        },
        VariableEnvironment: {
            EnvironmentRecord: {
                c: undefined
            }
            OuterLexicalEnvironment: null
        }
    }
{% endcodeblock %}

그리고 `multiply(20, 30)` 코드를 만난 순간 multiply 함수의 EC가 만들어지게 됩니다.

{% codeblock lang:JavaScript %}
    FunctionExecutionContext = {
        LexicalEnvironment: {
            EnvironmentRecord: {
                e: 20,
                f: 30
            }
            OuterLexicalEnvironment: GlobalLexicalEnvironment
        },
        VariableEnvironment: {
            EnvironmentRecord: {
                g: undefined
            }
            OuterLexicalEnvironment: GlobalLexicalEnvironment
        }
    }
{% endcodeblock %}

multiply 함수가 실행되면 g 변수에 값을 할당하고 계산 값을 리턴하게 됩니다.

{% codeblock lang:JavaScript %}
    FunctionExecutionContext = {
        LexicalEnvironment: {
            EnvironmentRecord: {
                e: 20,
                f: 30
            }
            OuterLexicalEnvironment: GlobalLexicalEnvironment
        },
        VariableEnvironment: {
            EnvironmentRecord: {
                g: 20
            }
            OuterLexicalEnvironment: GlobalLexicalEnvironment
        }
    }
{% endcodeblock %}

그리고 계산된 값은 c에 할당되게 되고 모든 코드 실행이 완료되면 프로그램이 종료됩니다.

## 5. 참고 사항

제가 참고한 글을 포함해 많은 글들이 EC가 생성되는 과정을 **Creation Phase와 Execution Phase**라는 단어를 이용해 설명하고 있습니다.
그런데 spec에서 아무리 검색을 해도 해당 단어들이 나오지 않았습니다. 이상하게 여기던 와중 stackoverflow의 글을 찾을 수 있었습니다.

{% blockquote stackoverflow https://stackoverflow.com/questions/62935334/creation-and-execution-phase-in-ecmascript-specification Creation and execution phase in ECMAScript Specification [closed] %}
    There are no "phases" in the spec and they're not called like that, but the concept still exists.
{% endblockquote %}

한 마디로, phase라는 단어를 이용하진 않았지만 개념적으로 존재한다는 의미였습니다. {% link MDN의 Hoisting 페이지 https://developer.mozilla.org/en-US/docs/Glossary/Hoisting %}에서도 creation and execution phases라는 단어를 쓴 것을 보면 이 단어들을 써서 설명을 해도 오류가 없을 듯 합니다.

## ※ 참고 링크

- {% link ECMAScript® 2015 Language Specification https://262.ecma-international.org/6.0 %}
- {% link Understanding Execution Context and Execution Stack in Javascript https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0 %}
- {% link [JS] 자바스크립트의 The Execution Context (실행 컨텍스트) 와 Hoisting (호이스팅) https://velog.io/@imacoolgirlyo/JS-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-Hoisting-The-Execution-Context-%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-6bjsmmlmgy %}
- {% link Execution Context https://medium.com/@snaag.dev/execution-context-d53a258c7415 %}
- {% link [JS] Execution Context(실행 컨텍스트) https://baeharam.netlify.app/posts/javascript/execution-context %}