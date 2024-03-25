# Rendering

### TL;DR

1. HTML을 파싱해 DOM 노드로 구성된 트리(**DOM**) 생성
2. CSS를 파싱해 CSS 노드로 구성된 트리(**CSSOM**) 생성
3. DOM과 CSSOM 트리를 결합해 **렌더 트리** 생성
4. 렌더 트리를 기반으로 **레이아웃/리플로우** 과정 실행
5. 각 노드를 화면에 **페인팅**

### DOM 생성

브라우저의 렌더링 엔진이 가장 먼저 수행하는 작업은 Document Object Model, DOM 트리를 생성하는 것이다. HTML 파일을 DOM 트리로 파싱하는 과정은 총 4단계로 구성된다.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

1. **바이트 인코딩**: 디스크 또는 네트워크에서 HTML의 원시 바이트를 읽고, 파일에 지정된 인코딩 방식(ex. UTF-8)에 따라 변환한다.
2. **토큰화**: 변환된 문자열을 HTML5 표준에 따라 고유한 토큰(ex. `<html>`, `<head>`)으로 변환한다.
3. **객체화**: 각 토큰을 속성이 정의된 객체로 변환한다.
4. **DOM 구성**: 원본 마크업의 포함 관계 등을 고려해 트리 구조를 생성한다.

#### 잘못 작성된 HTML

```html
<body>
<p>Front-end Developer
<span>Zero
```

위의 코드는 HTML 파일의 시작 지점인 `<html>` 태그를 정의하지 않았고, 닫는 태그(`</body>`, `</p>`)를 지정하지 않았다. 이는 문법적으로 틀린 코드이지만, HTML 파서는 자동으로 문법에 맞게 마크업을 수정한다. 위의 코드를 브라우저에서 실행시켰을 때의 모습이다.

```html
<html>
  <head></head>
  <body>
    <p>
      Front-end Developer <span>Zero</span>
    </p>
  </body>
</html>
```

#### 외부 자원

HTML 파서는 이미지, CSS 등의 논블로킹 자원을 발견하면, 해당 자원을 요청하고 구문 분석을 계속한다. 하지만 `<script>`  태그의 경우, [`async`나 `defer` 속성](https://ko.javascript.info/script-async-defer)이 정의되어 있지 않으면 구문 분석을 멈추고 해당 스크립트를 해석한다. 해석이 끝난 이후에 다시 구문 분석을 진행한다.

### CSSOM 생성

HTML 파싱 과정에서 발견한 CSS 또한 HTML과 유사한 파싱 과정을 거쳐 CSSOM(CSS Object Model)을 생성한다. 이 작업은 DOM을 생성하는 작업과 독립적으로 진행된다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### 재귀

CSSOM 생성 과정에서는, 브라우저에서 기본 제공하는 User Agent Stylesheet의 내용과 CSS 파일의 스타일을 포함한다. 그리고 일반적으로 적용된 스타일부터 시작해 구체적으로 적용된 스타일까지 재귀적으로 트리의 내용을 구성한다.

### 렌더 트리 생성

렌더링 엔진은 DOM 트리가 생성된 이후에, DOM 트리와 CSSOM 트리를 합쳐 렌더 트리를 생성한다. 렌더링 트리 구성 과정은 다음과 같다.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

1. DOM 트리의 루트에서부터 노드를 순회한다.
2. 화면에 표시되지 않는 노드를 생략한다. (ex. `display: none`, `<script>`, `<meta>`, ...)
3. 콘텐츠와 [캐스케이드 방식으로 생성된 스타일](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)을 기반으로 렌더 트리를 생성한다.

화면에 표시되지 않는 노드를 생략하기 때문에, DOM 트리와 렌더 트리는 1:1으로 매칭되지 않을 수 있다.

### 레이아웃 & 리플로우

렌더 트리를 생성하는 과정에서 각 노드의 스타일을 계산하였지만, 실제 디바이스의 뷰포트 내에서의 위치와 크기를 계산하지는 않았다. 이 과정을 레이아웃 또는 리플로우라 칭한다.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### 전역/증분 레이아웃

글꼴 크기를 변경하는 등의 전역적 스타일 변경, 뷰포트 크기가 변경되는 경우 렌더 트리를 전체적으로 다시 순회하며 전역 레이아웃 과정을 거친다. 그 외에 스타일이 변경되는 경우에는 비동기적으로 증분 레이아웃을 실행한다. 증분 레이아웃은 변경되어야 하는 노드에 한해 실행된다.

### 페인팅

페인팅 단계에서는, 렌더 트리의 레이아웃 과정을 기반으로 노드를 화면에 실제로 표출한다. 레이아웃과 마찬가지로 페인팅 과정에도 전역/증분 페인팅이 존재한다. 증분 페인팅에서는 변경되어야 하는 노드에 한해 페인팅 과정을 재실행한다.&#x20;

### 리렌더링

렌더링 엔진에서 수행하는 모든 연산은 리소스를 요구한다. 특히 레이아웃과 페인팅 과정의 경우, 자주 수행될 시 성능적으로 문제가 발생할 수 있다.

DOM을 직접적으로 조작하는 경우, 잦은 리렌더링을 초래하게 될 가능성이 커진다. 예를 들어, for문을 순회하며 새로운 노드를 추가할 때 순회한 횟수만큼 리렌더링이 발생된다. 이를 방지하기 위해 React, Vue와 같은 프론트엔드 라이브러리는 가상 DOM을 변경하는 방식을 택한다. 변경사항을 가상 DOM에 반영하고, DOM과 비교하여 실제로 변경된 부분만 DOM에 반영한다.

### 참고 문서

* [Critical Rendering Path | web.dev](https://web.dev/articles/critical-rendering-path)
* [How browsers work | web.dev](https://web.dev/articles/howbrowserswork)
* [웹페이지를 표시한다는 것: 브라우저는 어떻게 동작하는가 | mdn](https://developer.mozilla.org/ko/docs/Web/Performance/How\_browsers\_work)
* [Life of a Pixel | Steve Cobs | Chromium University](https://www.youtube.com/watch?v=K2QHdgAKP-s)
* [프론트엔드 개발자라면 알고 있어야 할 브라우저의 동작 과정 | 재그지그](https://wormwlrm.github.io/2021/03/27/How-browsers-work.html)
* [Parsing HTML documents | HTML Standard](https://html.spec.whatwg.org/multipage/parsing.html#parsing)
