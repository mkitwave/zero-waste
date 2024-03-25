# Event Loop

JavaScript는 싱글 스레드 언어이기 때문에 한 번에 하나의 작업만 처리할 수 있다. 하지만 대부분의 웹 어플리케이션에서는 네트워크 요청이나 이벤트 처리 같은 여러 개의 작업들을 동시에 처리한다. 이 문서에서는 JavaScript가 이러한 동시성을 이벤트 루프를 이용해 해결하는 방법을 설명한다.

### Call Stack

```log
RangeError: Maximum call stack size exceeded
```

JavaScript에서 무한 재귀 코드를 실행했을 때, 위와 같은 에러를 본 적이 있을 것이다. 여기서 언급된 'call stack'은, JavaScript 함수를 실행할 때 사용되는 스택 구조이다. 'stack' 이라는 네이밍에 걸맞게 LIFO(Last In First Out) 방식이다.&#x20;

JavaScript는 하나의 call stack만을 사용하기 때문에, 현재 stack에 추가된 모든 함수들이 실행될 때까지 다른 함수를 실행할 수 없다. 그렇다면 이미지 프로세싱, 네트워크 요청 등의 무거운 작업들이 call stack에 추가되었다고 가정해보자. 다른 함수들은 '블로킹'된 상태로 무거운 작업이 끝나기 전까지 실행되지 못할 것이다.&#x20;

### Callback Queue

이러한 문제를 JavaScript에서는 비동기 콜백으로 해결한다. 이 비동기 콜백이 쌓이는 곳이 callback queue로, task queue, microtask queue, animation frame 등으로 구성되어 있다. callback queue는 FIFO(First In First Out) 방식으로, 먼저 추가된 콜백이 먼저 제거된다.&#x20;

예시를 살펴보자.

```javascript
function a() {
    setTimeout(() => console.log('setTimeout a'), 1000);
}

function b() {
    console.log('b');
}

a();
b();
```

위 코드를 실행하면 console에 'b'가 먼저 출력되고 'setTimeout a'가 나중에 출력되는 것을 알 수 있다. 아래에 단계별로 call stack과 callback queue에서 어떤 일들이 발생하고 있는지 설명한다.

> 1. call stack에 `a`가 추가된다.&#x20;
> 2. call stack에 `a` 내부의 `setTimeout` 함수가 추가된다.
> 3. setTimeout은 브라우저에서 제공하는 Web API이기 때문에, 브라우저에서 타이머를 실행한다.
> 4. **(타이머의 완료와는 무관하게) call stack에서 `setTimeout` 함수가 제거된다.**
> 5. call stack에서 `a`가 제거된다.
> 6. call stack에 `b`가 추가된다.
> 7. call stack에 `b` 내부의 `console.log('b')` 함수가 추가된다.
> 8. `b` 내부의 `console.log('b')`를 실행한다.
> 9. call stack에서 `console.log('b')`, `b`가 제거된다.
> 10. 타이머가 완료되면 브라우저에서 `setTimeout`의 콜백 함수를 callback queue에 추가한다.
> 11. **call stack이 비어있으면, callback queue의 콜백 함수(`console.log('setTimeout a')`)를 call stack에 추가한다. (callback queue에서는 제거한다.)**
> 12. `console.log('setTimeout a')`를 실행한다.
> 13. call stack에서 `console.log('setTimeout a')`를 제거한다.

4번에서 말했듯, callback queue로 이동될 비동기 콜백은 다른 JavaScript 함수의 실행에 영향을 주지 않는다.

의문점이 드는 부분이 있다. 11번에서 call stack이 비어있으면, callback queue의 콜백 함수를 call stack에 추가하는 동작은 누가 담당하는 것일까?

### Event Loop

정답은 이 글의 주제인 이벤트 루프이다. 이벤트 루프는 **"call stack이 비면 callback queue에서 가장 오래된 콜백 함수를 call stack에 추가한다."** 라는 매우 단순한 알고리즘으로 동작한다. ([비주얼 예시](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif)) 하지만 위에서 언급했듯, callback queue에는 다양한 queue들이 존재하고 이들은 각각의 우선순위가 존재한다. 이벤트 루프는 이 우선순위에 따라 실행 순서를 결정하는데, 순서는 다음과 같다.

> 1. microtask queue(ex. Promise)
> 2. animation frame(ex. requestAnimationFrame)
> 3. task queue(ex. setTimeout, fetch)

<pre class="language-javascript" data-full-width="false"><code class="lang-javascript"><strong>setTimeout(() => {
</strong><strong>    console.log('task queue');
</strong><strong>}, 0);
</strong><strong>
</strong><strong>Promise.resolve().then(() => {
</strong><strong>    console.log('microtask queue');
</strong><strong>});
</strong><strong>
</strong><strong>requestAnimationFrame(() => {
</strong><strong>    console.log('animation frame');
</strong><strong>});
</strong></code></pre>

코드를 실행하면, console에 'microtask queue' -> 'animation frame' -> 'task queue'  순으로 출력된다.

### 번외: setTimeout의 비밀

`setTimeout(callback, 0)`은 0초 뒤 콜백 함수의 실행을 보장하지 않는다. 이유는 다음과 같다.

1. **call stack이 비어 있어야지만 call stack에 등록될 수 있다**: 위에서 설명했듯이, 우선순위가 더 높은 작업들이 먼저 실행된다.
2. &#x20;(브라우저의 경우) **5번째 중첩 타이머 실행 시, 타이머의 대기 시간이 4ms 이상으로 강제된다**: [HTML 표준으로 등록되어 있다.](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers)

#### 활용하기

동기적인 작업 또는 DOM 렌더링 이후에 호출되어야 하는 함수를 `setTimeout(callback, 0)`을 이용해 작성할 수 있다.

```javascript
setTimeout(() => {
  animation(); // render 이후에 실행된다.
}, 0);

render();
```



### 참고 문서

* [자바스크립트와 이벤트 루프 | NHN Cloud](https://meetup.nhncloud.com/posts/89)
* [대기 시간이 0인 setTimeout | javascript.info](https://ko.javascript.info/settimeout-setinterval#ref-24)
* [어쨌든 이벤트 루프는 무엇입니까? | Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
* [루프 속 | Jake Archibald | JSConf.Asia](https://www.youtube.com/watch?v=cCOL7MC4Pl0)

