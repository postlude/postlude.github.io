---
categories:
  - JavaScript
tags:
  - javascript
date: 2020-02-16 22:43:36
title: window와 BOM, DOM
updated:
---


javascript 관련된 여러가지 객체에 대해 한 번쯤 정리를 하려고 했었다.

처음에는 MDN의 관련 문서를 처음부터 끝까지 읽어보려고 시도하다가 이건 아니 것 같다는 느낌이 강하게 들면서(...)

개별 property나 세세하게 읽어보기 보다는 전체적인 개념을 한 번 정리해보고자 한다.

# 1. window

{% blockquote javascript.info https://ko.javascript.info/browser-environment 브라우저 환경과 다양한 명세서 %}
    자바스크립트는 본래 웹 브라우저에서 사용하려고 만든 언어입니다. 이후 진화를 거쳐 다양한 사용처와 플랫폼을 지원하는 언어로 변모하였습니다.

    자바스크립트가 돌아가는 플랫폼은 호스트(host) 라고 불립니다. 호스트는 브라우저, 웹서버, 심지어는 커피 머신이 될 수도 있습니다. 각 플랫폼은 해당 플랫폼에 특정되는 기능을 제공하는데, 자바스크립트 명세서에선 이를 호스트 환경(host environment) 이라고 부릅니다.

    호스트 환경은 랭귀지 코어(ECMAScript)에 더하여 플랫폼에 특정되는 객체와 함수를 제공합니다. 웹브라우저는 웹페이지를 제어하기 위한 수단을 제공하고, Node.js는 서버 사이드 기능을 제공해주죠.

    아래 그림은 호스트 환경이 웹 브라우저일 때 사용할 수 있는 기능을 개괄적으로 보여줍니다.

    {% asset_img 1.JPG %}

    최상단엔 window라 불리는 ‘루트’ 객체가 있습니다. window 객체는 2가지 역할을 합니다.

    1. 전역 객체 챕터에서 설명한 바와 같이, 자바스크립트 코드의 전역 객체입니다.
    2. '브라우저 창(browser window)'을 대변하고, 이를 제어할 수 있는 메서드를 제공합니다.
{% endblockquote %}

각각의 탭은 각각의 Window 객체를 가진다. JavaScript 코드가 실행되는 전역 window는 항상 코드가 실행되는 탭을 가리킨다.
몇몇 properties와 함수는 탭을 포함한 전체 창에 대해 적용된다.(ex. resizeTo(), innerHeight)

호스트에 따라 DOM과 BOM은 존재하지 않을수도 있다. 하지만 Javascript Core는 공통적으로 존재.

# 2. DOM

{% blockquote javascript.info https://ko.javascript.info/browser-environment#ref-502 문서 객체 모델(DOM) %}
    문서 객체 모델(Document Object Model, DOM)은 웹 페이지 내의 모든 콘텐츠를 객체로 나타내줍니다. 이 객체는 수정 가능합니다.
    document 객체는 페이지의 기본 ‘진입점’ 역할을 합니다. document 객체를 이용해 페이지 내 그 무엇이든 변경할 수 있고, 원하는 것을 만들 수도 있습니다.
{% endblockquote %}

{% asset_img 2.JPG %}

{% asset_img 3.JPG %}

document를 통해 문서(html)를 조작할 수 있음.

# 3. BOM

{% blockquote javascript.info https://ko.javascript.info/browser-environment#ref-503 브라우저 객체 모델(BOM) %}
    브라우저 객체 모델(Browser Object Model, BOM)은 문서(document) 이외의 모든 것을 제어하기 위해 브라우저(호스트 환경)가 제공하는 추가 객체를 나타냅니다.
    alert/confirm/prompt 역시 BOM의 일부입니다. 문서(document)와 직접 연결되어 있지 않지만, 사용자와 브라우저 사이의 커뮤니케이션을 도와주는 순수 브라우저 메서드이죠.
{% endblockquote %}

## 3.1 navigator

{% blockquote javascript.info https://ko.javascript.info/browser-environment#ref-503 브라우저 객체 모델(BOM) %}
    navigator 객체는 브라우저와 운영체제에 대한 정보를 제공합니다. 객체엔 다양한 프로퍼티가 있는데, 가장 잘 알려진 프로퍼티는 현재 사용 중인 브라우저 정보를 알려주는 navigator.userAgent와 브라우저가 실행 중인 운영체제(Windows, Linux, Mac 등) 정보를 알려주는 navigator.platform입니다.
{% endblockquote %}

{% asset_img 4.JPG %}

## 3.2 location

{% blockquote javascript.info https://ko.javascript.info/browser-environment#ref-503 브라우저 객체 모델(BOM) %}
    location 객체는 현재 URL을 읽을 수 있게 해주고 새로운 URL로 변경(redirect)할 수 있게 해줍니다.
{% endblockquote %}

{% asset_img 5.JPG %}

## 3.3 screen

{% blockquote www.zerocho.com https://www.zerocho.com/category/JavaScript/post/573b321aa54b5e8427432946 Window 객체와 BOM %}
    화면에 대한 정보를 알려줍니다. 너비(width), 높이(height), 픽셀(pixelDepth), 컬러(colorDepth), 화면 방향(orientation), 작업표시줄을 제외한 너비와 높이(availWidth, availHeight) 등이 있습니다. 화면 크기에 따라 다른 동작을 하고 싶을 때 사용합니다.
{% endblockquote %}

{% asset_img 6.JPG %}

## 3.4 history

{% blockquote MDN https://developer.mozilla.org/ko/docs/Web/API/History History %}
    History 인터페이스는 브라우저의 세션 기록, 즉 현재 페이지를 불러온 탭 또는 프레임의 방문 기록을 조작할 수 있는 방법을 제공합니다.
{% endblockquote %}

{% blockquote www.zerocho.com https://www.zerocho.com/category/JavaScript/post/573b321aa54b5e8427432946 Window 객체와 BOM %}
    history.pushState(객체, 제목, 주소)와 history.replaceState(객체, 제목, 주소)는 HTML5에서 추가되었는데요. 페이지를 이동하지 않고 단순히 주소만 바꿔줍니다. 대신 객체 부분에 페이지에 대한 정보를 추가할 수 있습니다.
{% endblockquote %}
