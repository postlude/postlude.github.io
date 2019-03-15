---
categories:
  - null
tags:
  - null
date: 2019-02-06 23:40:47
title: javascript-execution-context
updated:
---



JavaScript의 실행 컨텍스트(Execution Context)에 대해 정리해보고자 한다.

ECMAScript 2015 문서에 따르면 Excution Context(이하 EC)의 정의는 아래와 같다.

{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts Execution Contexts %}
    An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation. At any point in time, there is at most one execution context that is actually executing code. This is known as the running execution context. A stack is used to track execution contexts. The running execution context is always the top element of this stack. A new execution context is created whenever control is transferred from the executable code associated with the currently running execution context to executable code that is not associated with that execution context. The newly created execution context is pushed onto the stack and becomes the running execution context.
{% endblockquote %}

첫 문장부터 의문이 생겼다. **runtime evaluation**은 뭘까? 구글링을 통해 찾은 의미는 다음과 같았다.

{% blockquote rosettacode.org https://rosettacode.org/wiki/Runtime_evaluation Runtime evaluation %}
    Demonstrate a language's ability for programs to execute code written in the language provided at runtime.
{% endblockquote %}

즉, 실행 평가(runtime evaluation)란 어떤 코드가 런타임에 어떤 기능을 하는지 판단하는 것이다.

다시 EC로 돌아오면, EC의 역할은 이 실행 평가를 추적하는 것이다. 그 구조는 stack으로 되어 있고.
마지막 2문장을 보면 control이 실행 가능한 코드(executable code)로 옮겨가면 새로운 EC가 생성되고 stack에 push된다고 나와있다. 그리고 실행 중인 EC는 새로 생성된 EC가 된다고 한다.
문서에 나와있는 소스 코드의 종류는 다음과 같다.

{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-types-of-source-code Types of Source Code %}
    Global code is source text that is treated as an ECMAScript Script. The global code of a particular Script does not include any source text that is parsed as part of a FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.

    Eval code is the source text supplied to the built-in eval function. More precisely, if the parameter to the built-in eval function is a String, it is treated as an ECMAScript Script. The eval code for a particular invocation of eval is the global code portion of that Script.

    Function code is source text that is parsed to supply the value of the [[ECMAScriptCode]] and [[FormalParameters]] internal slots (see 9.2) of an ECMAScript function object. The function code of a particular ECMAScript function does not include any source text that is parsed as the function code of a nested FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.

    Module code is source text that is code that is provided as a ModuleBody. It is the code that is directly evaluated when a module is initialized. The module code of a particular module does not include any source text that is parsed as part of a nested FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.
{% endblockquote %}

이 기준으로 EC가 만들어진다는 것인데 Function code 부분을 보면 

{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts Execution Contexts %}
    An execution context contains whatever implementation specific state is necessary to track the execution progress of its associated code. Each execution context has at least the state components listed in Table 22.

    Table 22 —State Components for All Execution Contexts

    | Component | Purpose |
    |:-:|:-:|
    | code evaluation state | Any state needed to perform, suspend, and resume evaluation of the code associated with this execution context. |
    | Function | If this execution context is evaluating the code of a function object, then the value of this component is that function object. If the context is evaluating the code of a Script or Module, the value is null. |
    | Realm | The Realm from which associated code accesses ECMAScript resources. |
{% endblockquote %}




{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts Execution Contexts %}
    Execution contexts for ECMAScript code have the additional state components listed in Table 23.

    Table 23 — Additional State Components for ECMAScript Code Execution Contexts

    | Component | Purpose |
    |:-:|:-:|
    | LexicalEnvironment | Identifies the Lexical Environment used to resolve identifier references made by code within this execution context. |
    | VariableEnvironment | Identifies the Lexical Environment whose EnvironmentRecord holds bindings created by VariableStatements within this execution context. |

    The LexicalEnvironment and VariableEnvironment components of an execution context are always Lexical Environments. When an execution context is created its LexicalEnvironment and VariableEnvironment components initially have the same value.
    {% endblockquote %}



{% blockquote ECMAScript® 2015 Language Specification http://www.ecma-international.org/ecma-262/6.0/#sec-environment-records Environment Records %}
    There are two primary kinds of Environment Record values used in this specification: declarative Environment Records and object Environment Records. Declarative Environment Records are used to define the effect of ECMAScript language syntactic elements such as FunctionDeclarations, VariableDeclarations, and Catch clauses that directly associate identifier bindings with ECMAScript language values. Object Environment Records are used to define the effect of ECMAScript elements such as WithStatement that associate identifier bindings with the properties of some object. Global Environment Records and function Environment Records are specializations that are used for specifically for Script global declarations and for top-level declarations within functions.
{% endblockquote %}


