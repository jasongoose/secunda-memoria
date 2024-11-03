---
title: Synchronizing w/ Effects
date: 2024-11-04 08:20:12
categories:
  - Studies
  - React
  - Learn React
#tags:
---
React의 Effect를 사용하면 컴포넌트의 mount(첫 렌더링) 또는 리렌더링(commit 이후)될 때마다 dependency state 값의 변화에 맞춰 side-effect를 수행하거나 컴포넌트 unmount 또는 리렌더링 직전에 cleanup 작업을 수행할 수 있습니다.

## useEffect

Effect를 사용하려면 먼저 `useEffect` hook으로 Effect를 정의한 함수를 wrap하면 됩니다.

```jsx
import { useEffect } from "react";

function MyComponent() {
  useEffect(() => {
    // Effect는 commit 이후에 실행됩니다.
    // 컴포넌트가 렌더링될 때마다 useEffect 함수가 호출되면서 기능은 동일하지만 새로운 Effect가 등록됩니다.
  });
  return <div />;
}
```

컴포넌트의 렌더링 이후, 특정 dependency(state)들의 변화에 맞춰서도 Effect를 수행하려면 dependency array에 해당 state들을 전달하면 됩니다.

```jsx
useEffect(() => {
  // 첫 렌더링(mount) + 이후 리렌더링(commit)이 끝날 때마다 한번씩
});

useEffect(() => {
  // dependency가 없으므로 첫 렌더링(mount)이 끝났을 때만 한번
}, []);

useEffect(() => {
  // 첫 렌더링(mount) + 이후 리렌더링하면서 직전 snapshot의 dependency state들 중 하나라도 변한 경우 한번
}, [a, b]);
```

여기서 컴포넌트 context에서 정의된 `useRef`의 ref는 언제나 동일한 DOM 객체를 참조하기 때문에 dependency로 추가해도 Effect를 호출하지 않습니다.

단, 상위 컴포넌트에서 props로 전달된 ref 같은 경우, 변화여부를 하위 컴포넌트에서 확인할 수 없으므로 dependency로 추가할 수 있습니다.

## cleanup

Effect의 cleanup 함수를 사용하면 컴포넌트가 unmount되기 전, 그리고 dependecy state의 변화에 의한 리렌더링을 진행하면서 새 effect를 수행하기 직전에 기존 Effect를 중지하거나 무효화시키는 로직을 구현할 수 있습니다.

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
  return <h1>Welcome to the chat!</h1>;
}

/*
컴포넌트가 mount될 때 한번 connection(setup)이 일어나고,
컴포넌트가 unmount될 때 disconnect(cleanup)가 일어나고,
컴포넌트가 re-mount될 때 다시 connection(setup)이 일어납니다.

보통 React는 dev환경에서 Strict Mode를 적용하여 컴포넌트가 한번 mount되면
unmount한 뒤에 다시 mount를 수행합니다(prod에서는 한번만 mount). 

re-mount를 수행하면 unmount 이후, mount를 수행했을 때
Effect가 적절하게 cleanup된 뒤에 다시 정상적으로 수행되는지 여부를 확인할 수 있습니다!

따라서, re-mount를 거쳐도 앱이 정상적으로 동작하도록 개발해야 합니다.
*/
```

Effect cleanup은 다음과 같은 상황에서 활용될 수 있습니다.

1. mount시 `DOM.addEventListener`, unmount시 `DOM.removeEventListener`

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(e.clientX, e.clientY);
  }
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
}, []);
```

2. mount시 animation trigger, unmount시 animation reset

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

3. mount시 fetch, 리렌더링 또는 unmount시 기존 response 무효화

```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

mount되어 현재 userId state를 기준으로 fetchTodos를 수행하는데 중간에 userId 값이 변해서 리렌더링하거나 unmount되는 경우, 기존 fetch의 응답이 컴포넌트 상태에 영향이 가지 않도록 합니다.

useEffect에서 fetch한다면 기본적으로 caching이 안되므로 Next.js, Gatsby 등 프레임워크나 react-router+6.4, react-query, useSWR 같은 라이브러의 fetch api를 사용하는 것을 권장합니다.

URL이 바뀔 때마다 log를 POST하는 로직은 Effect보다는 route change event handler로 구현하는게 낫습니다.

## 참고자료

[Learn React](https://react.dev/learn)