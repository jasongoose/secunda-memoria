---
title: Referencing Values with Refs
date: 2024-11-04 08:15:06
categories:
  - Studies
  - React
  - Learn React
#tags:
---
컴포넌트 렌더링 사이에서 계속 유지하려는 값은 state 대신 ref 객체에 저장하면 됩니다.

## useRef

ref를 사용하려면 `useRef` hook에 initial value를 인자로 전달하면 되고, `ref.current` 속성에 접근하여 read/write하면 됩니다.

```jsx
import { useRef } from 'react';

const ref = useRef(0);
// initial value로는 모든 데이터 타입(심지어 함수까지)이 가능합니다.

// read
ref.current + ...;

// write
ref.current.~ = ~;
ref.current = {...};
```

`useRef`의 hook에서 반환되는 값은 JS 객체이고 아래와 같은 특징을 가집니다.

1. state와는 달리 새로운 객체로 할당되어도 컴포넌트 리렌더링을 trigger하지 않는다.
2. 첫 렌더링 때만 초기화되고, 컴포넌트가 리렌더링되어도 이전 값을 유지한다.

ref는 컴포넌트 렌더링 주기의 영향없이(escape hatch) 외부 API(Browser API)를 사용할 수 있다는 점에서 아래 값들을 저장하는데 활용됩니다.

1. unmount시 timer/interval을 clear하기 위한 `setTimeout`, `setInterval` id
2. `setTimeout`, `setInterval` 콜백에서 참조할 최신 state값
3. DOM element에 접근하기 위한 수단
4. 반환되는 JSX를 계산하는데 필요하지 않은 데이터

`ref.current` 값을 read/write하는 코드는 컴포넌트 렌더링 중 즉, 컴포넌트 함수의 context 상에 위치해서는 안됩니다.

state처럼 리렌더링을 trigger하지 않기 때문에 컴포넌트의 동작방식을 추적하는데 어려움을 주므로 보통 event handler에서 다룹니다.
