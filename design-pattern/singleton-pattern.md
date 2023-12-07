## Singleton 패턴

> JavaScript 기준으로 설명합니다.

## 용어

- **인스턴스**(Instance): 어떤 생성자를 기반으로 생성된 새로운 객체 ([mdn](https://developer.mozilla.org/en-US/docs/Glossary/Instance))
  - 생성자가 쿠키를 찍어낼 수 있는 쿠키 틀이라면, 인스턴스는 쿠키 틀에서 찍어낸 쿠키이다.
- **인스턴스화**(Instantiation): 생성자로부터 인스턴스를 생성하는 것
- **객체 리터럴**(Object literal): 중괄호(`{}`)를 사용해 객체를 초기화하는 방식 ([mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer))

## 의미와 특성

클래스의 인스턴스화를 단일 인스턴스로 제한하는 패턴이다.

싱글톤 패턴은 아래와 같은 특성을 갖는다.

- **단일 인스턴스가 보장**된다.
- 해당 인스턴스에 **쉽게 접근**할 수 있다.

## 구현 예시

### 싱글톤 패턴으로 커스텀 Logger 구현하기

```js
let instance;
let logCounter = 0;

class Logger {
  constructor() {
    if (instance) {
      throw new Error("중복으로 인스턴스를 생성할 수 없습니다.");
    }
    instance = this;
  }

  getInstance() {
    return this;
  }

  success(message) {
    ++logCounter;
    console.log(`SUCCESS(${logCounter}): ${message}`);
  }

  error(message) {
    ++logCounter;
    console.log(`ERROR(${logCounter}): ${message}`);
  }
}

const logger = Object.freeze(new Logger()); // logger 객체의 수정을 방지한다.
```

단일 인스턴스가 보장되기 때문에 `success`, `error` 메서드가 어디에서 호출되더라도 `logCounter` 값이 공유된다.

### 객체 리터럴로 구현하기

```js
let logCounter = 0;

const logger = {
  success(message) {
    ++logCounter;
    console.log(`SUCCESS(${logCounter}): ${message}`);
  },
  error(message) {
    ++logCounter;
    console.log(`ERROR(${logCounter}): ${message}`);
  },
};

Object.freeze(logger);
```

## 장점

- 단일 인스턴스를 강제하기 때문에 매번 새로운 인스턴스를 초기화하는 것보다 **메모리 공간이 절약**된다.

## 단점

- **단위 테스트가 어렵다.**
  - 테스트마다 인스턴스를 생성할 수 없기 때문에 하나의 인스턴스로 모든 테스트가 이루어져야 한다. 즉, 모든 테스트들은 이전 테스트에서 생성 또는 수정한 인스턴스를 수정해야 하고, 이렇게 환경에 강하게 결합된 테스트는 실행 순서 등에 따라 예기치 못한 동작을 할 가능성이 높다.
- **단일 책임 원칙에 위배된다.**
  - 단일 인스턴스를 보장하며, 인스턴스에 대해 전역 접근을 제공하는 두 기능을 동시에 하기 때문에, 단일 책임 원칙에 위배된다.
- **전역적으로 접근 가능하다.**
  - 이는 싱글톤 패턴의 특징이자 단점인데, `var`를 더 이상 사용하지 않는 이유에서 알 수 있듯 아무 곳에서나 변경 가능한 전역 상태는 복잡한 애플리케이션에서 데이터 흐름을 더 복잡하게 만든다.

## 참고 자료

- [Singleton pattern | Wikipedia](https://en.wikipedia.org/wiki/Singleton_pattern)
- [Singleton 패턴 | patterns.dev](https://patterns-dev-kr.github.io/design-patterns/singleton-pattern/)
