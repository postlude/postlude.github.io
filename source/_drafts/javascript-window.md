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

{% blockquote MDN https://developer.mozilla.org/en-US/docs/Web/API/Window Window %}
    DOM을 포함하는 객체. document property는 해당 window가 로드될 때의 DOM 문서를 가리킨다.
    전역 변수이며, JavaScript 코드가 실행되는 window를 나타낸다.
    
    각각의 탭은 각각의 Window 객체를 가진다. JavaScript 코드가 실행되는 전역 window는 항상 코드가 실행되는 탭을 가리킨다.
    즉, 몇몇 properties와 함수는 전체 window에 대해 적용된다.
{% endblockquote %}

MDN에 나와 있는 대략적인 설명은 위와 같다.

property와 method, event가 너무 많아서.. 더 이상 사용하지 않거나 표준이 아니라고 하는 등의 표시가 있는 것들은 제외하고

나머지만 간략하게만 정리.(대괄호 안의 문자는 spec status 값)

## 1. Properties

### 1.1. closed
[LS] 해당 window가 close되었는지 여부 판단. boolean 값.

### 1.2. console
[LS] Console 객체 반환. console.log() 의 그 console.

### 1.3. crypto
[REC] Crypto 객체를 반환. read only. 암호화와 관련있는 서비스에 사용된다..고 하는데 정확한 용도는 잘모르겠습니다.
MDN에 나와있는 예시는 `window.crypto.getRandomValues()`를 이용해 랜덤한 숫자를 생성하는 것입니다.

{% codeblock window.crypto lang:JavaScript %}
    var array = new Uint32Array(10);
    window.crypto.getRandomValues(array);
    // [3358826217, 1761377827, 2285082587, 3359671288, 3983454662, 2300567207, 2810238414, 493257368, 354263111, 2427745326]
    // 위와 같은 형태로 랜덤 숫자가 생성
{% endcodeblock %}

### 1.4. customElements
[LS] CustomElementRegistry 객체를 반환하는 read only property.
custom element 등록하거나 등록된 custom element의 정보를 읽어올 때 사용.
CustomElementRegistry.define() 함수를 통해 새로운 custom element 등록.

{% codeblock window.customElenents lang:JavaScript %}
    let customElementRegistry = window.customElements;
    customElementRegistry.define('my-custom-element', MyCustomElement);
{% endcodeblock %}

### 1.5. devicePixelRatio
[WD] 현재 표시 장치의 물리적 픽셀과 CSS 픽셀의 비율을 반환합니다. CSS 픽셀의 크기를 물리적 픽셀의 크기로 나눈 값으로 해석해도 됩니다.
또 다른 해석은, 하나의 CSS 픽셀을 그릴 때 사용해야 하는 장치 픽셀의 수라고 할 수 있습니다.

### 1.6. document
다른 문서에서 따로 설명

### 1.7. event
[LS] 현재 처리 중인 이벤트 반환. 이벤트 핸들러 밖에서는 undefined.
이벤트 핸들러에서의 event를 사용하는 것을 권장. 예측되는 결과가 아닌 결과를 반환하는 경우가 발생할 수 있음.

### 1.8. frameElement
[CR] window에 포함된 element(iframe 이나 object)를 반환.

{% asset_img 1.JPG %}

### 1.9. frames
[LS / REC] window 자기 자신을 반환하되 array처럼 [i] 형태로 i번째 frame에 접근 가능.
- window.frames === window : true
- window.frames[0] === document.getElementsByTagName("iframe")[0].contentWindow : true

### 1.10. history
다른 문서에서 따로 설명

### 1.11. indexedDB / localStorage / sessionStorage
다른 문서에서 따로 설명

### 1.12. innerHeight / innerWidth
[WD] read only property 로 window 높이(너비) 값을 반환. 수평(수직) 스크롤바가 있다면 포함.
- window 사이즈에서 수평 스크롤 바와 border 를 제외한 값을 알고 싶다면 root html element 에서 clientWidth() 를 사용.
- 사이즈 변경을 위해서는 resizeBy() 나 resizeTo() 메소드를 사용.

### 1.13. length
[LS / REC] frame(혹은 iframe)의 개수 반환.

### 1.14. location
다른 문서에서 따로 설명

### 1.15. name
[LS / REC] Gets/sets the name of the window.

### 1.16. navigator
다른 문서에서 따로 설명

### 1.17. opener
window.open() 이나 link의 target attribute를 이용해 현재 window를 오픈한 window를 가리킴.
없으면 null.

### 1.18. outerHeight / outerWidth
[WD] 브라우저 바깥쪽(전체) window 가로/세로 길이.

### 1.19. scrollX / scrollY / pageXOffset / pageYOffset
[WD] 스크롤의 수평/수직 위치를 나타내는 값.
크로스 브라우징을 위해서는 pageXOffset/pageYOffset 을 사용.
pageXOffset : scrollX의 alias
pageYOffset : scrollY의 alias

### 1.20. parent
[LS] 현재 window 또는 subframe(iframe, object, 또는 frame)의 부모를 가리킴. parent가 없으면 자기 자신.

### 1.21. performance
[REC] 현재 document로부터 수집된 performance 정보를 가진 객체를 리턴.
(Performance Timeline, High Resolution Time, Navigation Timing, User Timing, Resource Timing)

### 1.22. screen
다른 문서에서 따로 설명

### 1.23. screenX / screenY / screenLeft / screenTop
[WD] 화면 상의 좌상단으로부터 수평 / 수직으로 얼마나 떨어져 있는지를 나타내는 값.
screenLeft : alias of the older Window.screenX property
screenTop : alias of the older Window.screenY property
screenTop, screenLeft 는 IE만 support.

### 1.24. self
[LS / REC] window 자기 자신을 가리킴.
(worker : background task that can be created via script, which can send messages back to its creator)

{% codeblock window.self lang:JavaScript %}
    var w1 = window;
    var w2 = self;
    var w3 = window.window;
    var w4 = window.self;
    // w1, w2, w3, w4 all strictly equal, but only w2 will function in workers
{% endcodeblock %}

### 1.25. status
[LS] 원래는 하단 상태 바에 텍스트를 출력하는 용도로 사용했으나 현재는 아무 효과 없음.

### 1.26. top
[LS / REC] window 계층 구조에서 가장 최상위 window를 반환.

### 1.27. window
[LS / REC] 자기 자신 window를 가리킴.

<hr />

## 2. Methods

### 2.1. alert()
alert 창 노출. alert가 닫힐 때까지 모든 인터페이스에 접근 불가.
Chrome 46.0(글 작성 기준 현재 79.0)부터, 이 메소드는 iframe에서 사용할 경우 그 iframe의 sandbox 속성에 allow-modal이 없으면 작동하지 않음.

### 2.2. atob(), btoa()
atob : base-64로 encoding 된 문자열 데이터를 decoding.
btoa : binary string을 base-64로 인코딩된 ASCII 문자열로 변환.

### 2.3. setInterval() / clearInterval() / setTimeout() / clearTimeout()
setInterval : 반복적으로 어떠한 함수를 실행. intervalID(0이 아닌 숫자 값) 를 리턴
clearInterval : intervalID를 parameter로 받아 setInterval 설정한 내용을 clear.
setTimeout : timer 종료 후에 함수 실행. timeoutID 리턴.
clearTimeout : intervalID를 parameter로 받아 setTimeout으로 실행되기 전 취소.

setTimeout과 setInterval은 같은 id pool을 사용하므로 clearTimeout과 clearInterval을 교환해서 사용은 가능.
하지만 명확하게 하기 위해서 그런식의 사용은 하지 말 것.

### 2.4. blur()
Shifts focus away from the window.

### 2.5. close()
현재 window 닫음. window.open()을 이용한 script로 연 window만 닫을 수 있으며, 그렇지 않을 경우 아래와 유사한 메시지 출력.
{% blockquote %}
    Scripts may not close windows that were not opened by script.
{% endblockquote %}

HTMLIFrame​Element​.content​Window 로 리턴된 window 객체에 대해서는 효과 없음.

### 2.6. confirm()
ok / cancel 2개의 버튼이 있는 확인 창 출력.
파라미터로 message를 출력하며, 리턴 값으로 ok를 눌렀을 때 true, cancel을 눌렀을 때 false 반환.

### 2.7. createImageBitmap()
주어진 이미지로부터 bitmap 생성.
edge, IE 미지원.

Parameters
- image
An image source, which can be an img, SVG image, video, canvas, HTMLImageElement, SVGImageElement, HTMLVideoElement, HTMLCanvasElement, Blob, ImageData, ImageBitmap, or OffscreenCanvas object.
- sx
The x coordinate of the reference point of the rectangle from which the ImageBitmap will be extracted.
- sy
The y coordinate of the reference point of the rectangle from which the ImageBitmap will be extracted.
- sw
The width of the rectangle from which the ImageBitmap will be extracted. This value can be negative.
- sh
The height of the rectangle from which the ImageBitmap will be extracted. This value can be negative.

Return
A Promise which resolves to an ImageBitmap object containing bitmap data from the given rectangle.

### 2.8. focus()
bring the window to the front. 유저 세팅으로 인해 실패할수도 있음.

### 2.9. getComputedStyle()
element에 적용된 모든 CSS를 포함하는 객체를 리턴. 반환된 객체에 getPropertyValue() 함수를 이용해 CSS 설정 값을 읽을 수 있음.

{% codeblock window.getComputedStyle() lang:JavaScript %}
    let para = document.querySelector('p');
    let compStyles = window.getComputedStyle(para);
    compStyles.getPropertyValue('font-size');
    compStyles.getPropertyValue('line-height');
{% endcodeblock %}

### 2.10. getSelection()
유저가 선택한(드래그한) 영역이나 현재 캐럿(^) 위치의 정보를 가지고 있는 Selection 객체 반환.
빈 문자열을 더하거나 toString() 이용해서 현재 선택 영역의 문자열을 읽을 수 있음.
보여지고 있지 않은 iframe(display: none 이 세팅되어 있는 등의 이유로)에서 호출되면 FireFox는 null을 리턴하고,
다른 브라우저는 Selection.type이 None인 Selection 객체 반환.
Document.getSelection() 와 동일.
Document.activeElement 는 focus된 element를 반환(selectino과의 차이점)

### 2.11. matchMedia()

