## Observer 패턴

> JavaScript 기준으로 설명합니다.

## 용어

- 옵저버(Observer): 특정 객체를 구독하는 주체
- 옵저버블(Observable): 구독 가능한 객체

## 의미와 특성

상태 변화가 있을 때마다 옵저버블이 해당 옵저버블에 등록된 각 옵저버에 이벤트를 전파하는 패턴이다.
Observer 패턴은 아래와 같은 특성을 가진다.

- 옵저버블과 옵저버는 일대다 종속성을 갖는다.
- 옵저버블의 상태가 변경되면 등록된 옵저버에 자동으로 이벤트가 전파되어야 한다.

## RxJS를 활용한 Github User Search 구현

[RxJS](https://rxjs.dev/)는 Observer 패턴, 이터레이터 패턴, 함수형 프로그래밍을 조합한 ReactiveX 라이브러리이다.

([코드](https://github.com/mkitwave/rxjs-github-user-search/blob/main/index.html), [입문서](https://chasethestar.gitbook.io/ko.learn-rxjs/rxjs/concepts/rxjs-primer))

## 장점

- 옵저버와 옵저버블의 관계를 느슨하게 유지했을 때, 관심사 분리를 강제할 수 있다.
- RxJS의 경우, 복잡한 비동기 처리를 다루기 쉬워진다.

## 단점

- 옵저버가 복잡해지거나, 제때 구독 해제하지 않았을 때 성능 이슈가 발생할 수 있다.
- RxJS의 경우, 학습 허들이 높은 편이다.

## 참고 자료

- [Observer pattern | Wikipedia](https://en.wikipedia.org/wiki/Observer_pattern)
- [Observer 패턴 | patterns.dev](https://patterns-dev-kr.github.io/design-patterns/observer-pattern/)
