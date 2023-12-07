## Prototype 패턴

> JavaScript 기준으로 설명합니다.

## 의미

객체를 복제하여 생성함으로써 새로운 객체를 만들어내는 패턴이다.

## JavaScript Prototype

JavaScript에서는 모든 객체가 자체적으로 프로토타입을 가진다. (프로토타입 기반 언어라고 불리는 이유이다.) 객체를 생성할 때 해당 객체의 프로토타입을 기반으로 새로운 객체를 만들 수 있으며, 이는 클래스 객체 생성 시에도 마찬가지이다. 클래스는 프로토타입을 더 쉽게 다룰 수 있도록 구문적 설탕(syntactic sugar) 역할을 할 뿐 내부적으로는 프로토타입 기반의 상속을 사용한다.

기존 객체의 속성은 객체 생성자의 `prototype` 프로퍼티에 정의된다. `Object.getPrototypeOf()`, `Object.setPrototypeOf()` 메소드를 통해 `prototype`을 조회·변경할 수 있다.

### Prototype Chain

객체가 프로퍼티나 메서드에 접근하려고 할 때, 해당 객체에 그 프로퍼티나 메서드가 없다면 자동으로 상위 프로토타입 객체에서 검색한다. 이를 프로토타입 체인이라고 한다.

[이 문서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#different_ways_of_creating_and_mutating_prototype_chains)에서 프로토타입 체인을 생성하는 다양한 방법을 비교하는데, 아래는 `Object.create()`로 생성한 예시이다.

```js
const dog = {
  bark() {
    console.log(`멍멍!`);
  },
};

const myPet = Object.create(dog);

myPet.bark(); // → 멍멍!

console.log(Object.keys(myPet)); // → []
console.log(Object.keys(Object.getPrototypeOf(myPet))); // → ['bark']
```

## 장점

- 프로토타입 체인을 통해 대상 객체에 프로퍼티가 직접 선언되어있지 않아도 되므로, 메모리를 절약할 수 있다.

## 단점

> 실제로 코드가 실행되는 동안 `Object.prototype`을 수정한다는 것은 성능따위 창문밖으로 던져버린다는 것을 뜻합니다. 절대 하지마세요! - [JavaScript engine fundamentals: optimizing prototypes](https://shlrur.github.io/javascripts/javascript-engine-fundamentals-optimizing-prototypes/)

- [JavaScript 공식 문서에서도 강력히 경고](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)하고 있는 내용으로, 언어적으로 `prototype`의 수정에 있어 성능적, 범위적 리스크가 크다. `Object.create()`를 사용하도록 하자.

## 참고 자료

- [Object prototypes | mdn](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)
- [Inheritance and the prototype chain | mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
- [Prototype 패턴 | patterns.dev](https://patterns-dev-kr.github.io/design-patterns/prototype-pattern/)
