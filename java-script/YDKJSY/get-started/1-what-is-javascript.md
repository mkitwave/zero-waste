# JavaScript가 무엇인가요?

> You Don't Know JS Yet을 읽고 정리한 글입니다. ([원문](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/get-started/ch1.md))

## 이름

JavaScript의 이름은 마케팅적 측면에서 지어진 것이다. 당시 메이저 언어였던 **Java**와, 경량 프로그램을 지칭하는 용어로 많이 사용되었던 **Script**를 합쳐 JavaScript가 되었다.
Java와 완전히 관련이 없는 것은 아니긴 하다. (구문적 유사점, Oracle과의 법적 관계)
Oracle이 "JavaScript"라는 이름의 상표를 소유하고 있기 때문에, TC39에서 지정하고 ECMA 표준 기구에서 공식화한 명칭은 **ECMAScript**이다. 2016년부터는 개정 연도를 붙여 ECMAScript 2019(약칭 ES2019) 등으로 쓰여진다.

## 언어 명세

TC39에서 JS의 공식 명세를 관리하고, 이를 표준 기구인 ECMA에 제출한다. TC39에 제안하고, 제안서에 대해 토론하는 것은 누구나 할 수 있지만, 공식적으로 "0단계" 제안서로 간주되기 위해서는 TC39 위원회 멤버의 동의가 필요하다. 제안이 "4단계"에 다다르면 다음 연도 JS 개정에 포함될 수 있다. 모든 제안은 [TC39의 GitHub 레포](https://github.com/tc39/proposals)에서 확인할 수 있다.

### 웹이 JS를 지배한다

JS를 실행하는 환경은 다양하지만, 웹 브라우저에서 JS가 구현되는 방식이 가장 중요하다고 할 수 있다. 대부분의 경우 명세에 정의된 JS와 브라우저의 JS 엔진에서 실행되는 JS는 동일하지만, 몇 가지 차이점이 있다.
JS 엔진에서는 레거시 웹 페이지가 손상될 가능성을 고려해 명세를 따르지 않을 수 있고, TC39에서 이를 역추적해서 웹의 현실에 맞게 변경하기도 한다. (ex. Array에 `contains()`메소드를 추가할 예정이었으나 `includes()`로 네이밍 변경)
반대로 TS39에서 JS 엔진에서 명세를 준수하지 않을 가능성을 감수하고 변경을 진행할 때도 있다. ([참고](https://tc39.es/ecma262/multipage/additional-ecmascript-features-for-web-browsers.html))

### 모두 JS인 건 아니다

`alert()`, `console.log()`, `fetch()` 등은 JS 명세에 정의되어있지 않지만, JS 문법을 따르기 때문에 JS처럼 보인다. 이 동작들은 JS 엔진을 실행하는 환경에 의해 제어된다.

### 개발자 도구 콘솔 != JS 환경

개발자 도구의 콘솔은 엄격하게 준수된 JS 동작을 하는 환경이라고 보기 어렵다. 호이스팅의 작동 방식, 전역 변수 생성 스코프 등의 차이점이 존재하기 때문에, **"JS 친화적인 환경"** 정도로만 보는 것이 좋다.

## 다중 패러다임

JS는 절차적, 객체 지향, 함수형 프로그래밍 등의 패러다임을 모두 지원한다. 서로 다른 패러다임을 혼합해서 사용하는 방식도 가능하다.

## 상위/하위 호환성

JS는 **하위 호환성**(Backwards compatibility)을 지원하지만, 상위 호환성(Forwards compatibility)을 지원하지 않는다.
즉, **이전 버전의 JS를 최신 버전의 JS 엔진에서 정상적으로 실행하는 것을 보장하지만, 최신 버전의 JS를 이전 버전의 JS 엔진에서 정상적으로 실행하는 것은 보장되지 않는다**는 것이다.
드물게 이 원칙을 깨는 변경들이 있지만, 감수할만한 범위인지 체크하고 신중하게 처리한다.
반대로 HTML, CSS는 상위 호환성을 지원하고 하위 호환성을 지원하지 않는다. 구형 브라우저에서 최신 버전의 HTML, CSS 명세를 사용할 경우 인식되지 않는 HTML, CSS를 건너뛰는 식으로 처리된다.

### 트랜스파일링

위에서 설명한 것처럼 이전 버전의 JS 엔진에서는 최신 버전의 JS 문법이 호환되지 않는다. 이 문제에 대한 해결책으로 트랜스파일링(일반적으로 [Babel](https://babeljs.io/))을 사용한다. 트랜스파일링 도구를 통해 개발자는 깔끔하고 새로운 문법으로 코드를 작성하는 데 집중하고, 이전 버전의 JS 엔진을 고려하기 위한 수고를 덜 수 있다.

### 폴리필

호환성 문제가 문법에 관련된 것이 아니라 새롭게 추가된 API와 관련되었다면, 이전 버전의 엔진에서 누락된 API에 대한 정의를 제공하여 이미 기본적으로 정의되어 있는 것처럼 동작하게 하는 작업이 필요하다. 이를 폴리필이라고 한다. 일반적으로 폴리필은 JS 엔진이 최신 버전이라고 판단(API가 누락되지 않은)되는 경우, 엔진의 기본 동작을 수행하고 그 외의 경우 폴리필을 정의해 안정적으로 동작하도록 한다. (ex. [resize-observer-polyfill](https://github.com/que-etc/resize-observer-polyfill/blob/master/src/index.js))

## 인터프리트(interpreted), 파싱(parsed), 컴파일(compiled)

인터프리티드 언어는 일반적으로 줄 단위 하향식으로 실행되기 때문에, 5번째 줄에 에러가 존재해도 1-4번째 줄까지는 정상적으로 실행된다. 반면에 파싱된 언어는 코드를 분석하는 전처리 과정을 거치기 때문에, 같은 상황이 발생되면 파싱 과정에서 에러가 발생한다.
책에서는 파싱된 언어가 사실상 컴파일드 언어라고 주장한다. 모든 컴파일드 언어는 파싱 과정을 거치고, 보통의 파싱된 언어 또한 실행 가능한 형태의 코드를 생성하기 때문이다.
**JS 코드 또한 실행되기 전에 파싱되므로, 컴파일드 언어이자 파싱된 언어**라는 것이 결론이다.

### 웹 어셈블리(WASM)

WASM은 JS 파싱/컴파일의 지연 문제를 해결하고 JS 외의 언어를 JS 엔진에서 실행할 수 있는 형태로 변환하기 위해 만들어진 언어이다. WASM으로 인해 JS 명세를 결정할 때 다른 언어 생태계의 요구에 치우치지 않고 판단할 수 있게 되었으며, 다른 언어도 웹(꼭 웹만은 아님)에서 실행할 수 있게 되었다.
책에서는, WASM으로 인해 다른 언어가 JS를 대체할 거라고 주장하는 사람들에게 그렇지 않다고 강력히 주장한다.

## 엄격 모드

엄격 모드는 좋은 품질의 코드와 나은 성능을 위해 ES5에서 도입된 기능이다. 이는 기본값이 아니지만, 기본값처럼 사용해야 한다. `use strict`를 파일 상단에 추가하거나, 스코프 내부 상단에 추가(마이그레이션 상황 외에 권장되지 않음)하면 엄격 모드가 적용된다.

## 결론

JS는 TC39에서 만들어지고 ECMA에서 호스팅하는 ECMAScript 표준의 구현체이다. 브라우저, Node.js와 같은 기타 JS 환경에서 실행된다.

JS는 다중 패러다임 언어로, 개발자가 절차적, 객체 지향, 함수형 등 다양한 주요 패러다임의 개념을 혼합하고 조합할 수 있는 구문과 기능을 갖추고 있다.

JS는 컴파일드 언어로, 이는 JS 엔진을 포함한 도구가 프로그램을 실행하기 전에 프로그램을 처리하고 검증하는 것을 의미한다.