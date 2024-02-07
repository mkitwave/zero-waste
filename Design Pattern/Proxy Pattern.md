> JavaScript 기준으로 설명합니다.

## 의미와 특성

설정, 조회 등의 기본적인 객체 동작을 재정의하는 Proxy 객체를 활용한 패턴이다. 어떤 객체를 사용하고자 할 때, 객체를 직접적으로 참조하는 것이 아니라 Proxy 객체를 통해 해당 객체에 접근한다.

프록시 패턴은 아래와 같은 특성을 갖는다.

- **원본 객체에 대한 접근을 제어한다.**
- JavaScript에서는 후술할 **`Proxy` 객체**를 통해 주로 구현한다.

## JavaScript Proxy

### [`Proxy`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

특정 객체를 감싸 객체에 가해지는 작업을 중간에서 가로채는 내장 객체이다. `new Proxy(target, handler);` 형태로 생성한다. 여기서 `target`은 원본 객체이고,
`handler`는 프록시의 동작을 정의하는 객체이다. 빈 `handler`로 생성된 프록시 객체는 원본 객체와 동일한 동작을 하며, `get`, `set` 등의 메소드를 `handler`에 추가하여 원본 객체의 속성에 접근하거나 수정할 수 있다. `handler`는 원본 객체에 대한 호출을 잡아낸다는 의미에서 **트랩**이라고 불리기도 한다.

생성 시 주의할 점이 몇 가지 있다.

- [프록시 객체는 원본 객체의 private 속성에 접근할 수 없다.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#no_private_property_forwarding)
- [엄격 모드에서 `set()` 메소드 사용 시 falsy한 값을 반환하면 TypeError가 발생한다.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/set#return_value)

### [`Reflect`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

`Proxy`를 더 간단하게 생성할 수 있게 하는 내장 객체이다. `Proxy`와 달리 함수 객체가 아니므로 생성자로 사용되지 않는다. (`new` 연산자를 통해 생성하지 않는다.) `Proxy handler`의 `get()`, `set()` 등의 기본 메소드를 갖고 있으나, `Proxy`는 메소드를 직접 호출할 수 없는 것에 반해 `Reflect`는 가능하다.

`Proxy handler` 내부 동작들을 대체하는 예시를 살펴보자.

- `get()` 메소드: `obj[prop]` -> `Reflect.get(obj, prop)`
- `set()` 메소드: `obj[prop] = value` -> `Reflect.set(obj, prop, value)`
  더 많은 예시는 [이 문서](https://ko.javascript.info/proxy#ref-274)에서 확인할 수 있다.

## 구현 예시

### 프록시 패턴으로 유효성 검사 구현하기

```js
const profile = {
  displayName: "Zero",
  age: 20,
};

const validateProfile = (prop, value) => {
  if (prop === "displayName") {
    if (value.length < 2) {
      throw new Error(`이름을 2자 이상 입력해주세요.`);
    }
  } else if (prop === "age") {
    if (value < 5) {
      throw new Error(`5세 이상만 가입할 수 있어요.`);
    }
  }

  return true;
};

const profileProxy = new Proxy(profile, {
  get: (object, prop) => {
    if (!object[prop]) {
      throw new Error(`존재하지 않는 속성이에요.`);
    } else {
      console.log(`${prop}: ${object[prop]}`);
    }
  },
  set: (object, prop, value) => {
    if (!object[prop]) {
      throw new Error(`존재하지 않는 속성이에요.`);
    }

    if (validateProfile(prop, value)) {
      object[prop] = value;
      console.log(`${prop} 속성이 ${value}로 변경되었어요.`);

      return true;
    }
  },
});

// throw 시 이후 코드는 실행되지 않지만, 가독성을 위해 단순 나열했습니다.

// 조회
profileProxy.description; // → Error: 존재하지 않는 속성이에요.
profileProxy.displayName; // → displayName: Zero

// 수정
profileProxy.displayName = "!"; // → Error: 이름을 2자 이상 입력해주세요.
profileProxy.displayName = "제로"; // → displayName 속성이 제로로 변경되었어요.

profileProxy.age = 3; // → Error: 5세 이상만 가입할 수 있어요.
profileProxy.age = 21; // → age 속성이 21로 변경되었어요.
```

### Reflect를 활용해 구현하기

```js
// ...

const profileProxy = new Proxy(profile, {
  get: (object, prop) => {
    if (!Reflect.get(object, prop)) {
      throw new Error(`존재하지 않는 속성이에요.`);
    } else {
      console.log(`${prop}: ${Reflect.get(object, prop)}`);
    }
  },
  set: (object, prop, value) => {
    if (!Reflect.get(object, prop)) {
      throw new Error(`존재하지 않는 속성이에요.`);
    }

    if (validateProfile(prop, value)) {
      Reflect.set(object, prop, value);
      console.log(`${prop} 속성이 ${value}로 변경되었어요.`);

      return true;
    }
  },
});

// 조회
profileProxy.displayName; // → displayName: Zero

// 수정
profileProxy.displayName = "제로"; // → displayName 속성이 제로로 변경되었어요.
profileProxy.age = 21; // → age 속성이 21로 변경되었어요.
```

[이 문서](https://github.com/mikaelbr/awesome-es2015-proxy#modules)에서 프록시를 활용해 구현된 다양한 모듈들을 확인할 수 있다. 개인적으로 [python-range](https://github.com/michal-perlakowski/range)가 가장 흥미로웠다.

## 장점

- 원본 객체에 대한 접근을 제어하는 특성상 가상 프록시(ex. 이미지 지연 로드), 보호 프록시(ex. 특정 객체에 대한 접근 제한) 등의 다양한 구현 상황에 적용할 수 있다.

## 단점

- 객체 생성 시 한 단계를 더 거치게 되므로, 빈번하게 생성되어야 하는 객체에 사용되었을 때 원본 객체를 직접 사용할 때보다 성능이 저하된다.
- 프록시 객체의 메서드 내에서 복잡한 작업이 발생하면 성능이 저하될 수 있다. ([참고 문서](http://thecodebarbarian.com/thoughts-on-es6-proxies-performance))

## 참고 자료

- [Proxy pattern | Wikipedia](https://en.wikipedia.org/wiki/Proxy_pattern)
- [Proxy 패턴 | patterns.dev](https://patterns-dev-kr.github.io/design-patterns/proxy-pattern/)
- [프록시 패턴에 대하여 | 코딩팩토리](https://coding-factory.tistory.com/711)