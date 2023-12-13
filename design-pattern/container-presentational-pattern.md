## Container/Presentational 패턴

> React 기준으로 설명합니다.

## 용어

- **관심사 분리**(separation of concerns, SoC): 프로그램을 하나의 단일 블록으로 작성하지 않고 작은 조각으로 나누어 각각 단순한 개별 작업을 완료하도록 만드는 작업. SOLID 원칙 중 2개가 이 개념에서 파생되었다.(단일 책임, 인터페이스 분리)

## 의미

Hook이 존재하기 전, 어플리케이션이 커졌을 때 컴포넌트 간의 의존도가 높아져 재사용이 어려웠고, 이러한 문제를 해결하기 위해 로직과 View 레이어를 나눠 관심사를 분리하기 위한 패턴이다.

## Container/Presentation 컴포넌트

### Container 컴포넌트

- **동작하는 부분을 담당**한다.
- 내부에 presentational 컴포넌트와 container 컴포넌트를 모두 포함할 수 있지만, 일부 래핑용 div를 제외하고는 자체적인 DOM 마크업과 스타일이 없다.
- presentational 컴포넌트 또는 다른 container 컴포넌트에 데이터와 동작을 제공한다.

### Presentational 컴포넌트

- **보여지는 부분을 담당**한다.
- 내부에 presentational 컴포넌트와 container 컴포넌트를 모두 포함할 수 있으며, 보통 자체적인 DOM 마크업과 스타일이 있다.
- store 등의 앱의 나머지 부분에 대한 종속성이 없다.
- 데이터 로드 또는 뮤테이션 코드가 없다.
- props를 통해서만 데이터와 콜백을 수신한다.
- 자체 상태를 거의 갖지 않는다.(있더라도 데이터가 아닌 UI 상태)

## 장점

- 컴포넌트로부터 복잡하고 stateful한 로직을 분리하게 해주어, 관심사 분리 측면에서 효과적이다.
- Presentation 컴포넌트는 앱의 비즈니스 로직을 수정하지 않으므로, 다양한 목적으로 재사용할 수 있다. 또한 요구되는 데이터를 props에 넘겨주기만 하면 되므로 Storybook 테스트 등에서 이점을 갖는다.

## 단점

- Container/Presentational 패턴을 처음 소개한 Dan Abramov는 현 시점에서 이 패턴을 사용하지 말라고 언급하였다. 이유는, 임의로 컴포넌트를 분리하지 않아도 **Hook**이 동일한 일을 해주기 때문이다.

## 참고 자료

- [Container/Presentational 패턴 | patterns.dev](https://patterns-dev-kr.github.io/design-patterns/container-presentational-pattern/)
- [Presentational and Container Components | Dan Abramov](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
