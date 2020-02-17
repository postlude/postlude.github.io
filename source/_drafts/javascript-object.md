---
categories:
  - null
tags:
  - null
date: 2020-02-16 22:43:36
title: title
updated:
---

한 번쯤 정리할 필요가 있어서 정리하고자 한다.

## 1.Window

{% blockquote MDN https://developer.mozilla.org/en-US/docs/Web/API/Window Window %}
    DOM을 포함하는 객체. document property는 해당 window가 로드될 때의 DOM 문서를 가리킨다.
    전역 변수이며, JavaScript 코드가 실행되는 window를 나타낸다.
    
    각각의 탭은 각각의 Window 객체를 가진다. JavaScript 코드가 실행되는 전역 window는 항상 코드가 실행되는 탭을 가리킨다.
    즉, 몇몇 properties와 함수는 전체 window에 대해 적용된다.
{% endblockquote %}

MDN에 나와 있는 대략적인 설명은 위와 같다.

property와 method, event가 너무 많아서.. 더 이상 사용하지 않거나 표준이 아니라고 하는 등의 표시가 있는 것들은 제외하고

나머지만 간략하게만 정리.

### 1.1. Properties

#### 1.1.1. closed
해당 window가 close되었는지 여부 판단. boolean 값.

#### 1.1.2. console
Console 객체 반환. console.log() 의 그 console.

#### 1.1.3. crypto
Crypto 객체를 반환. read only. 암호화와 관련있는 서비스에 사용된다..고 하는데 정확한 용도는 잘모르겠습니다.
MDN에 나와있는 예시는 `window.crypto.getRandomValues()`를 이용해 랜덤한 숫자를 생성하는 것입니다.

{% codeblock window.crypto lang:JavaScript %}
    var array = new Uint32Array(10);
    window.crypto.getRandomValues(array);
    // [3358826217, 1761377827, 2285082587, 3359671288, 3983454662, 2300567207, 2810238414, 493257368, 354263111, 2427745326]
    // 위와 같은 형태로 랜덤 숫자가 생성
{% endcodeblock %}

#### 1.1.4. customElements 
CustomElementRegistry 객체를 반환하는 read only property.
custom element 등록하거나 등록된 custom element의 정보를 읽어올 때 사용.
CustomElementRegistry.define() 함수를 통해 새로운 custom element 등록.

{% codeblock window.customElenents lang:JavaScript %}
    let customElementRegistry = window.customElements;
    customElementRegistry.define('my-custom-element', MyCustomElement);
{% endcodeblock %}

#### 1.1.5. devicePixelRatio
현재 표시 장치의 물리적 픽셀과 CSS 픽셀의 비율을 반환합니다. CSS 픽셀의 크기를 물리적 픽셀의 크기로 나눈 값으로 해석해도 됩니다.
또 다른 해석은, 하나의 CSS 픽셀을 그릴 때 사용해야 하는 장치 픽셀의 수라고 할 수 있습니다.

#### 1.1.6. document
다른 문서에서 따로 설명

#### 1.1.7. event
현재 처리 중인 이벤트 반환. 이벤트 핸들러 밖에서는 undefined.
이벤트 핸들러에서의 event를 사용하는 것을 권장. 예측되는 결과가 아닌 결과를 반환하는 경우가 발생할 수 있음.

#### 1.1.8. frameElement
window에 포함된 element(iframe 이나 object)를 반환.

{% asset_img 1.JPG %}

#### 1.1.9. frames
window 자기 자신을 반환하되 array처럼 [i] 형태로 i번째 frame에 접근 가능.
- window.frames === window : true
- window.frames[0] === document.getElementsByTagName("iframe")[0].contentWindow : true

#### 1.1.10. history
다른 문서에서 따로 설명

#### 1.1.11. indexedDB / localStorage / sessionStorage
다른 문서에서 따로 설명

#### 1.1.12. innerHeight / innerWidth
read only property 로 window 높이(너비) 값을 반환. 수평(수직) 스크롤바가 있다면 포함.
- window 사이즈에서 수평 스크롤 바와 border 를 제외한 값을 알고 싶다면 root html element 에서 clientWidth() 를 사용.
- 사이즈 변경을 위해서는 resizeBy() 나 resizeTo() 메소드를 사용.

#### 1.1.13. length
frame(혹은 iframe)의 개수 반환.

#### 1.1.14. location
다른 문서에서 따로 설명

#### 1.1.15. name
Gets/sets the name of the window.

#### 1.1.16. navigator
다른 문서에서 따로 설명

#### 1.1.17. opener
window.open() 이나 link의 target attribute를 이용해 현재 window를 오픈한 window를 가리킴.
없으면 null.

#### 1.1.18. outerHeight / outerWidth
브라우저 바깥쪽(전체) window 가로/세로 길이.

#### 1.1.19. scrollX / scrollY / pageXOffset / pageYOffset
pageXOffset : scrollX의 alias
pageYOffset : scrollY의 alias
스크롤의 수평/수직 위치를 나타내는 값.
크로스 브라우징을 위해서는 pageXOffset/pageYOffset 을 사용.

#### 1.1.20. parent
현재 window 또는 subframe(iframe, object, 또는 frame)의 부모를 가리킴. parent가 없으면 자기 자신.

#### 1.1.21. performance
현재 document로부터 수집된 performance 정보를 가진 객체를 리턴.
(Performance Timeline, High Resolution Time, Navigation Timing, User Timing, Resource Timing)

#### 1.1.22. screen
다른 문서에서 따로 설명

#### 1.1.23. screenX / screenY / screenLeft / screenTop
screenLeft : alias of the older Window.screenX property
screenTop : alias of the older Window.screenY property
screenTop, screenLeft 는 IE만 support.
화면 상의 좌상단으로부터 수평 / 수직으로 얼마나 떨어져 있는지를 나타내는 값.

#### 1.1.24. scrollbars
scrollbars 객체 반환.

#### 1.1.25. self
window 자기 자신을 가리킴.
(worker : background task that can be created via script, which can send messages back to its creator)

{% codeblock window.self lang:JavaScript %}
    var w1 = window;
    var w2 = self;
    var w3 = window.window;
    var w4 = window.self;
    // w1, w2, w3, w4 all strictly equal, but only w2 will function in workers
{% endcodeblock %}

#### 1.1.26. status
원래는 하단 상태 바에 텍스트를 출력하는 용도로 사용했으나 현재는 아무 효과 없음.

#### 1.1.27. statusbar
statusbar 객체 반환.

#### 1.1.28. toolbar
toolbar 객체 반환.

#### 1.1.29. top
window 계층 구조에서 가장 최상위 window를 반환.

#### 1.1.30. window
자기 자신 window를 가리킴.

### 1.2. Methods



## 2. Document
## 3. Location
## 4. History
## 5. Navigator