---
title: Lifecyle of Reactive Effects
date: 2024-11-04 08:11:56
categories:
  - Studies
  - React
  - Learn React
#tags:
---
## 컴포넌트 lifecycle

### mount

최초로 DOM에 반영됩니다.

### update(re-rendering)

보통 사용자 이벤트에 의한 state, props의 변화로 새로운 snapshot을 생성합니다.

### unmount

DOM에서 제거됩니다.

## Effect lifecycle

Effect는 현재 state와 props 또는 이들로부터 계산된 reactive value에 맞춰서 컴포넌트의 외부 시스템과의 setup(동기화 수행)과 cleanup(동기화 중단)을 수행합니다.

### onMount

Dependency 존재여부와 상관없이 처음으로 Effect를 수행합니다.

### onUpdate

리렌더링 중 dependency가 하나라도 변했다면 이전 Effect를 cleanup하고 현재 denpendency를 기준으로 Effect를 다시 수행합니다.

depenpency의 변화여부는 `Object.is` 함수를 사용합니다.

### onUnmount

컴포넌트가 DOM에서 사라지기 전에 현재 Effect를 cleanup합니다.

## Reactive values

컴포넌트 함수 블럭 내부에서 선언한 props, state를 포함한 변수들은 모두 컴포넌트 update시 재연산되는 성질이 있어서 reactive value라고 합니다.

Effect 내부에서 사용된 reactive value들은 모두 dependency로 등록하도록 linter가 강제합니다.

`window`와 같은 전역객체는 컴포넌트 rendering cycle 외부에서 mutate되므로 컴포넌트 리렌더링을 trigger하지 않습니다.

또한 useRef의 `ref.current` 값도 리렌더링없이 값을 추적하기 위해서 의도적으로 mutatble하므로 리렌더링을 trigger하지 않습니다.

컴포넌트 lifecylce을 따르지 않는 mutable value들의 상태를 subscribe/unsubscribe할 때는 `useSyncExternalStore` hook을 사용하면 됩니다.

## 참고자료

[Learn React](https://react.dev/learn)