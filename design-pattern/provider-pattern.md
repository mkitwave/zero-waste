## Provider 패턴

> React 기준으로 설명합니다.

## 용어

- **Prop drilliing**: React에서 하위 컴포넌트로 데이터를 전달하는 기본적인 방식은 props를 이용하는 것이다. 하지만 데이터를 전달해야 하는 하위 컴포넌트가 많거나, 데이터를 제공하는 상위 컴포넌트로부터 멀리 떨어져 있을 때 코드가 장황해지고 불편해질 수 있다. 보통 이러한 상황을 의미한다.

## 의미

Prop drilliing을 방지하기 위해, props를 전달하지 않고 하위 트리에 데이터를 전달하는 방식이다.
([Context | React 공식 문서](https://react.dev/learn/passing-data-deeply-with-context))

## 구현 예시

### Context Provider로 다크 모드 구현

```jsx
// ThemeContext.tsx

import { createContext } from "react";

const ThemeContext = createContext("light");

// context를 바로 export해서 사용하기보단, hook으로 래핑해서 쓰는 것이 일반적이다.
export const useThemeContext = () => {
  const theme = useContext(ThemeContext);

  if (!theme) {
    throw new Error("useThemeContext는 ThemeProvider 안에서 사용되어야 해요.");
  }

  return theme;
};

// 개인적으로 Provider도 래핑하는 것을 선호한다.
export const ThemeProvider = ({ theme, children }) => {
  return (
    <ThemeContext.Provider value={theme}>{children}</ThemeContext.Provider>
  );
};
```

## 장점

- 데이터를 제공하는 Provider와 소비하는 Consumer(위의 코드에서는 `useThemeContext`를 사용하는 컴포넌트)가 각자의 역할에 집중할 수 있다.
- Prop drilling을 방지할 수 있다.
- React에 내장되어 있는 기능이기 때문에 별도의 라이브러리 설치가 필요하지 않다.

## 단점

- 데이터 흐름이 명확하게 보이지 않는다.
- React에서 Context를 참조하는 모든 컴포넌트는 Context 값이 변경되었을 때마다 리렌더링되므로, Provider를 적절히 분리하지 않으면 성능 문제가 발생할 수 있다.

React 공식 문서에서는 Context를 대체할 수 있는 두 가지 방식을 제안하며, 무분별한 Context 사용을 지양하라고 경고한다.

- Props를 통해 전달하는 방식부터 시작하라.
- 컴포넌트를 분리해 `children`으로 전달하라. (ex. `<Layout posts={posts} />` → `<Layout><Posts posts={posts} /></Layout>`.)

## 참고 자료

- [Provider 패턴 | patterns.dev](https://patterns-dev-kr.github.io/design-patterns/provider-pattern/)
