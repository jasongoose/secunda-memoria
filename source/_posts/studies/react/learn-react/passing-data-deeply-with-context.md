---
title: Passing Data Deeply w/ Context
date: 2024-11-04 08:12:51
categories:
  - Studies
  - React
  - Learn React
#tags:
---
공통된 상위 컴포넌트에서 props를 통해 여러 하위 컴포넌트들로 데이터를 전달하면 중간에 거쳐야 하는 컴포넌트마다 일일이 props를 추가/수정해야 하고 이로 인해 컴포넌트의 재사용성을 떨어뜨리는 [props drilling](../../patterns/container_presenter.md#props-drilling) 문제가 생깁니다.

## Context API

React는 teleport 방식으로 상위 컴포넌트에서 깊숙히 위치한 하위 컴포넌트로 데이터를 전달하는 Context API를 제공합니다.

context를 생성하려면 `createContext` 함수를 default value와 함께 호출하면 됩니다.

```jsx
// LevelContext.js
import { createContext } from "react";

export const LevelContext = createContext(0);

// context로부터 전달받은 값이 없을 때 default value를 사용합니다.
// createContext를 호출할 때마다 별개의 context가 생성됩니다.
```

context를 적용할 때는 상위 컴포넌트(Provider)를 `Context.Provider`로 wrapping 하고 내려보낼 데이터를 `value` props에 명시합니다.

```jsx
import { LevelContext } from "./LevelContext.js";

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

context를 사용하는 자식/자손 컴포넌트(Consumer)는 `useContext` hook으로 전달된 데이터에 접근합니다.

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
	return {...};
}

// 만일 다수 상위 컴포넌트들이 동일한 context provider인 경우,
// consumer는 가장 가까운 context provider에서 전달한 value를 사용합니다.
```

provider도 또한 consumer이므로 상위 provider에서 전달받은 데이터를 가공하여 다시 context로 전달할 수 있습니다.

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

Context API는 다음과 같이 활용할 수 있습니다.

- [Theme provider w/ styled-component](https://styled-components.com/docs/advanced#theming)
- 현재 로그인한 사용자 계정 정보
- [Routing w/ react-router](https://reactrouter.com/en/main/routers/create-browser-router)
- State management w/ useReducer + useContext
