---
title: React Fiber Architecture란?
date: 2024-11-04 21:23:01
categories:
  - Studies
  - React
  - Fiber
#tags:
---
## Review

### What is reconciliation?

React에서 특정 state update(이하 update)는 기본적으로 전체 앱으로 하여금 리렌더링을 일으키는데 굉장히 많은 컴포넌트들로 구성된 큰 규모의 앱이라면 좋지 않은 성능의 원인이 될 수 있습니다.

그래서 이를 위한 최적화 기법인 Reconciliation을 제공합니다.

Reconciliation은 state update 전후로 생성된 VDOM의 node(React Element, 이하 react-el)들을 비교하여 새로운 화면을 그리기 위해서 필요한 diff를 계산하는데 여기서 2가지 규칙을 따릅니다.

- 다른 타입을 가진 react-el이 렌더링하는 경우 완전히 다른 트리로 간주하여 더 이상 diff를 계산하지 않고 기존 트리를 unmount한다.
- 배열형 react-el의 diff 연산은 `key`를 기준으로 한다.

### Reconciliation versus rendering

React에서 reconciliation과 rendering(=commit)을 개별 phase로 구분합니다.

그래서 동일한 reconciler를 사용하면서 다양한 renderer(ReactNative, react-pdf 등)를 plugin처럼 사용할 수 있는 이점이 있습니다.

### Scheduling

[React's Design Principles](https://legacy.reactjs.org/docs/design-principles.html#scheduling)에는 scheduling 관련해서 아래와 같이 설명합니다.

> In its current implementation React walks the tree recursively and calls render functions of the whole updated tree during a single tick. However in the future it might start delaying some updates to avoid dropping frames.
>
> This is a common theme in React design. Some popular libraries implement the "push" approach where computations are performed when the new data is available. React, however, sticks to the "pull" approach where computations can be delayed until necessary.
>
> React is not a generic data processing library. It is a library for building user interfaces. We think that it is uniquely positioned in an app to know which computations are relevant right now and which are not.
>
> If something is offscreen, we can delay any logic related to it. If data is arriving faster than the frame rate, we can coalesce and batch updates. We can prioritize work coming from user interactions (such as an animation caused by a button click) over less important background work (such as rendering new content just loaded from the network) to avoid dropping frames.

여기서 "work"는 임의의 컴포넌트의 리렌더링으로 수행할 연산들을 가리키는데 여기에 Effect도 포함됩니다.

위 인용구에서 말하고자 하는 바는 3가지로 정리할 수 있습니다.

- UI 상의 모든 update들을 동기적으로 처리할 필요가 없고 이는 오히려 frame drop으로 좋지 않은 UX를 줄 수 있다.
- 서로 다른 종류의 work들은 개별 우선순위를 부여한다.
- push 기반 접근방식을 따르면 app이 개별 work의 처리 순서를 직접 결정해야 하지만 pull 기반의 접근방식으로 따르면 프레임워크에 그대로 맡길 수 있다.

## What is a fiber?

위에서 소개한 React의 scheduling은 아래와 같은 조건을 충족해야만 합니다.

- work를 중지했다가 다시 재개할 수 있다.
- work의 타입별로 우선순위를 부여할 수 있다.
- 이전에 완료된 work를 그대로 재사용할 수 있다.
- 더 이상 필요하지 않은 작업은 폐기할 수 있다.

우선 컴포넌트는 아래와 같은 함수로 표현할 수 있습니다.

```js
v = f(d);
```

그래서 React 앱을 렌더링한다는 것은 곧 함수 안에서 중첩된 다른 함수들을 호출한다는 점을 의미하기도 합니다.

기존의 함수들은 [call stack](https://en.wikipedia.org/wiki/Call_stack)에 수행할 작업을 반영한 stack frame을 push되면 동기적으로 처리된 이후에 pop됩니다.

하지만 UI를 렌더링하는 컴포넌트 함수는 기존의 다른 함수들처럼 동기적으로 처리한다면 짧은 시간 안에 많은 작업들이 한꺼번에 몰리는 상황에 frame drop 현상이 발생할 수 있습니다.

현대 브라우저에서 렌더링 관련해서 작업에 우선순위를 부여할 수 있는 [`requestIdleCallback`](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback)과 [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)을 지원합니다.

`requestIdleCallback`은 animation frame의 렌더링 시점들 사이의 여유가 있는 기간에 실행할 우선순위가 낮은 함수를 지정할 수 있고 `requestAnimationFrame`은 다음 animation frame 렌더링 시점에 실행할 우선순위가 높은 함수를 지정할 수 있습니다.

![Frame Lifecycle](/images/frame_cycle.jpeg)

React의 scheduling 구현에 위 api들을 활용하려면 UI 렌더링 작업을 더 작은 단위로 나눠야하고 call stack에서 우선순위에 따라 call stack frame을 interrupt할 수 있도록 제어할 필요가 있습니다.

Fiber는 React 컴포넌트에 최적화된 stack을 위한 virtual stack frame으로 heap 메모리에 저장되어 원하는 시점에 실행할 수 있어서 concurrency와 error boundary를 구현할 수 있도록 도와줍니다.
