# Memoization

React에서는 상위 컴포넌트가 리렌더링되거나, 해당 컴포넌트의 상태가 변경된 경우에 리렌더링을 수행한다.&#x20;

### 불필요한 리렌더링

```jsx
const App = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <Description />
      <CounterButton
        count={count}
        onClick={() => setCount((prev) => prev + 1)}
      />
    </div>
  );
};

const CounterButton = ({ count, onClick }) => {
  return <button onClick={onClick}>Clicked {count} times!</button>;
};

const Description = () => {
  return <p>Click the button</p>;
};
```

위의 코드는 버튼을 클릭했을 때 `count`가 1씩 더해지는 간단한 예제이다. 버튼을 클릭하면 `App`, `CounterButton`, `Description` 컴포넌트가 모두 리렌더링된다.

그러나 `Description` 컴포넌트가 다시 리렌더링되어야 할 필요가 있을까? `Description` 컴포넌트는 `count`의 변경에 관계없이 동일한 결과로 렌더링될 것이다. 만약 `Description` 컴포넌트가 많은 하위 컴포넌트 또는 로직을 포함하고 있어 리렌더링하는 데 많은 리소스가 소요된다면, 불필요한 리렌더링에 의한 성능 문제가 발생할 수 있다.

이러한 '불필요한 리렌더링' 문제를 해결하기 위해 React에서는 컴포넌트, 값, 함수를 메모이제이션하는 `memo`, `useMemo`, `useCallback` API를 제공한다.

### memo

> 컴포넌트의 props가 변경되지 않은 경우, 컴포넌트 리렌더링을 건너뛸 수 있게 한다.\
> \- [React 공식 문서](https://react.dev/reference/react/memo)

[불필요한 리렌더링](memoization.md#undefined)의 예제에서, `count` 상태의 변경으로 인한 `Description` 컴포넌트의 리렌더링을 막는 데 `memo`를 사용할 수 있다.

```jsx
const Description = memo(() => {
  return <p>Click the button</p>;
});
```

`Description` 컴포넌트를 위와 같이 변경하면, `CounterButton`을 클릭해도 `Description` 컴포넌트가 리렌더링되지 않는다.

#### memo는 만능이 아니다

컴포넌트를 `memo`로 감쌌음에도 불구하고, 리렌더링되는 몇 가지 사례가 있다.

1. **props가 변경되었을 때**

```jsx
const CounterButton = memo(({ count, onClick }) => {
  return <button onClick={onClick}>Clicked {count} times!</button>;
});
```

`CounterButton` 컴포넌트는 `count` 상태가 변경되면 리렌더링된다. `onClick` 함수가 변경되었을 때에도 마찬가지이다. 당연한 것처럼 보이지만 이는 의도되지 않은 동작을 발생시킬 수 있다.

React는 얕은 비교를 통해 props를 비교한다. 얕은 비교로 객체나 배열, 함수를 비교하면, 이전의 값과 동일하더라도 참조가 동일한지에 대한 여부를 따진다. 그러므로 상위 컴포넌트에서 렌더링할 때마다 객체, 배열, 함수를 다시 생성한다면 memo된 컴포넌트 또한 props가 변경된 것으로 간주해 리렌더링을 발생시킨다.

이를 방지하기 위해 `useMemo`, `useCallback`을 사용하거나, 이후 설명할 `arePropsEqual` 파라미터를 사용할 수 있다.

2. **컴포넌트 내부의 상태가 변경되었을 때**
3. #### `arePropsEqual` 값이 false일 때 <a href="#memo" id="memo"></a>

`memo` HOC(고차 컴포넌트)에는 두 번째 인자로 `arePropsEqual` 파라미터를 추가할 수 있다.

```jsx
const Chart = memo(({ points }) => {
    ...
}, (oldProps, newProps) => {
  return (
    oldProps.points.length === newProps.points.length &&
    oldProps.points.every((oldPoint, index) => {
      const newPoint = newProps.points[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
});
```

위 예제의 `arePropsEqual` 함수는 `points` 객체의 값이 변경되었는지를 확인해, 변경되지 않았다면 `true`, 변경되었으면 `false`를 반환한다. 참조가 아닌 값을 비교하기 때문에 `points`가 부모 컴포넌트에서 재생성되어 참조가 변경되어도, 값이 변하지 않았다면 `Chart` 컴포넌트는 리렌더링되지 않는다.

하지만 이 방법 또한 만능은 아니다. React 공식 문서에서는 `arePropsEqual` 사용 시 모든 props를 비교해야 하며, prop의 데이터 구조 깊이가 제한되어 있지 않다면 깊은 비교를 수행하지 말라고 권고하고 있다. ([공식 문서](https://react.dev/reference/react/memo#specifying-a-custom-comparison-function))

4. **컴포넌트 내부에서 Context를 사용하고 있을 때**

```jsx
const CounterButton = memo(({ onClick }) => {
  const { count } = useContext(globalContext);
  
  return <button onClick={onClick}>Clicked {count} times!</button>;
});
```

위 예제에서 `globalContext`의 `count` 상태가 변경된다면 `CounterButton`이 리렌더링된다. 여기서 주의할 점은 `globalContext`에 다른 상태가 존재하고, 이 상태가 변경된다면 `CounterButton`에서 그 상태를 사용하고 있지 않음에도 리렌더링된다는 것이다.&#x20;

이를 방지하는 방법은 `useContext` 사용부를 부모 컴포넌트로 이동시켜, `count`를 props로 전달하는 것이다.

### useMemo

> 리렌더링 간에 계산 결과를 캐싱한다.\
> \- [React 공식 문서](https://react.dev/reference/react/useMemo#my-calculation-runs-twice-on-every-re-render)

<pre class="language-jsx"><code class="lang-jsx">const Chart = ({ data, ... }) => {
    const points = calculatePoints(data);

    const memorizedPoints = useMemo(
        () => calculatePoints(data),
        [data]
<strong>    );
</strong><strong>    
</strong><strong>    return &#x3C;ChartInner points={points} />
</strong>}

const ChartInner = memo(({ points }) => ...)

</code></pre>

위의 예제에서 `Chart`가 리렌더링 될 때마다 `points`는 다시 계산된다. `calculatePoints`가 무거운 로직일 때, 불필요한 재계산을 막기 위해 `useMemo`를 사용할 수 있다. `memorizedPoints`는 의존성 배열에 포함된 `data`를 `Object.is`(얕은 비교)로 비교해 변화를 감지하고, `data`가 변경되었을 때에 한해 캐싱을 무효화해 계산을 시행한다.

#### memo와 함께 사용하기

[memo는 만능이 아니다](memoization.md#memo-1) 섹션에서 설명했듯이, `memo`로 감싼 컴포넌트가 객체나 배열인 prop을 포함하고 있을 때 참조가 변경되면 리렌더링이 발생한다.&#x20;

위 예제의 `points`를 prop으로 넘겨줄 경우, `Chart`가 리렌더링 될 때마다 `points`의 참조가 변경되고, 값이 변경되지 않았음에도 `ChartInner`에 연쇄적인 리렌더링이 발생한다. 반면에 `memorizedPoints`를 넘겨준다면 `data`가 변경되지 않는 한 동일한 참조를 갖기 때문에 불필요한 리렌더링이 발생하지 않는다.

#### 주의사항

* useMemo는 Strict Mode에서 두 번 호출된다.
* memo와 마찬가지로 얕은 비교로 의존성 배열을 비교하기 때문에, 객체 등을 의존성 배열에 넣을 때 정상적으로 메모이제이션 되어있는지 확인해야 한다.
* 의존성 배열을 정의하지 않으면 컴포넌트가 리렌더링될 때마다 다시 호출된다.

### useCallback

> 리렌더링 간에 함수 정의를 캐싱한다.\
> \- [React 공식 문서](https://react.dev/reference/react/useCallback)

`useMemo`가 변수를 캐싱한다면, `useCallback`은 함수를 캐싱한다.

```jsx
const handleClick1 = useCallback(() => {
    setCounter(prev => prev + 1);
}, []);

const handleClick2 = useMemo(() => {
    return () => setCounter(prev => prev + 1);
}, []);
```

위 예제에서 두 함수의 작동은 동일하다. 하지만 함수를 캐싱할 때는 함수용으로 제공되는 `useCallback`을 사용하는 것이 가독성 면에서 좋다.

`useMemo`와 동일하게 `memo`로 감싸진 컴포넌트의 불필요한 리렌더링을 방지하는 데 사용될 수 있으며, 의존성 배열의 얕은 비교를 주의해야 한다.

### 전부 메모해야 할까?

`memo`, `useMemo`, `useCallback`에 대해 알아보았다. 이 최적화 기법들은 언제 사용하는 것이 좋을까? 렌더링이 자주 발생하는 컴포넌트? 계산 로직이 복잡한 컴포넌트? 판단하기 어려우니 전부 다?

메모이제이션의 기준에 대해서는 여러 의견이 존재하고, 어떤 것이 섣불리 좋다라고 말하기 어려운 주제이기도 하다. React 팀에서는 **메모이제이션 또한 비용**이기 때문에, 불필요한 메모이제이션을 대체하는 방식들을 제안한다.

> **몇 가지 원칙을 따르면 많은 메모이제이션을 불필요하게 만들 수 있습니다.**
>
> 1. 컴포넌트가 다른 컴포넌트를 시각적으로 감싸고 있다면 [JSX를 자식으로 받게](https://ko.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children) 하세요. 감싸는 컴포넌트가 자신의 상태를 업데이트하면, React는 자식들은 리렌더링할 필요가 없다는 것을 알게 됩니다.
> 2. 가능한 한 로컬 상태를 선호하고, [컴포넌트 간 상태 공유](https://ko.react.dev/learn/sharing-state-between-components)를 필요 이상으로 하지 마세요. 폼이나 항목이 호버되었는지와 같은 일시적인 상태를 트리의 상단이나 전역 상태 라이브러리에 유지하지 마세요.
> 3. [렌더링 로직을 순수하게 유지](https://ko.react.dev/learn/keeping-components-pure)하세요. 컴포넌트를 리렌더링하는 것이 문제를 일으키거나 눈에 띄는 시각적인 형체를 생성한다면, 그것은 컴포넌트의 버그입니다! 메모이제이션을 추가하는 대신 버그를 해결하세요.
> 4. [상태를 업데이트하는 불필요한 Effects](https://ko.react.dev/learn/you-might-not-need-an-effect)를 피하세요. React 앱에서 대부분의 성능 문제는 Effects로부터 발생한 연속된 업데이트가 컴포넌트를 계속해서 렌더링하는 것이 원인입니다.
> 5. [Effects에서 불필요한 의존성을 제거](https://ko.react.dev/learn/removing-effect-dependencies)하세요. 예를 들어, 메모이제이션 대신 객체나 함수를 Effect 안이나 컴포넌트 외부로 이동시키는 것이 더 간단한 경우가 많습니다.
>
> \- React 공식 문서

메모이제이션을 대체할 수 있는 방법에 대해 설명한 다른 좋은 글도 첨부한다.

* [Before You memo() | overreacted](https://overreacted.io/before-you-memo/)
* [One simple trick to optimize React re-renders | Kent C. Dodds](https://kentcdodds.com/blog/optimize-react-re-renders)



### 참고 문서

* [Why React Re-Renders | Josh W Comeau](https://www.joshwcomeau.com/react/why-react-re-renders)
* [Understanding useMemo and useCallback | Josh W Comeau](https://www.joshwcomeau.com/react/usememo-and-usecallback)
* [모던 리액트 Deep Dive | 김용찬](https://m.yes24.com/Goods/Detail/123161563)
