---
title: Say no to "flickering" UI, useLayoutEffect, painting and browsers story
date: 2024-11-04 21:21:28
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
DOM measurement(size, position 등의 정보)를 기반으로 HTML element에 변화를 주는 방법과 이를 위한 `useLayoutEffect` hook, 그리고 SSR 대응방식에 대해서 정리합니다.

## What is the problem with useEffect?

실무에서 viewport의 width에 맞춰서 nav 메뉴들의 일부만을 노출시키는 로직을 개발해야 하는 상황이라면 어떻게 구현할까요?

![Collapsed](/images/collapsed.png)

우선 화면에 렌더링이 되고나서 nav 메뉴 container(`div.navigation`)의 width와 개별 nav 메뉴의 width 합을 계산해야 하므로 관련 로직을 `useEffect`에 정의할 수 있습니다.

```jsx
const Component = ({ items }) => {
  const ref = useRef(null);

  useEffect(() => {
    const div = ref.current;
    const { width } = div.getBoundingClientRect();

    // convert div's children into an array
    const children = [...div.childNodes];

    // all the widths
    const childrenWidths = children.map(
      (child) => child.getBoundingClientRect().width
    );
  }, [ref]);

  return (
    <div className="navigation" ref={ref}>
      {items.map((item) => (
        <a href={item.href}>{item.name}</a>
      ))}
      <button id="more">...</button>
    </div>
  );
};
```

`useEffect` hook에 구현한 로직을 바탕으로 가장 마지막에 보여줄 수 있는 nav 메뉴의 index를 구하는 함수인 `getLastVisibleItem`을 정의할 수 있습니다.

```jsx
useEffect(() => {
  const itemIndex = getLastVisibleItem(ref.current);
});
```

그럼 아래와 같이 effect 단에서 컴포넌트 state를 계산한 index로 업데이트하면 리렌더링을 trigger하여 의도한 결과를 만들 수 있습니다.

```jsx
const Component = ({ items }) => {
  const [lastVisibleMenuItem, setLastVisibleMenuItem] = useState(-1);

  useEffect(() => {
    const itemIndex = getLastVisibleItem(ref.current);
    // update state with the actual number
    setLastVisibleMenuItem(itemIndex);
  }, [ref]);

  // render everything if it's the first pass and the value is still the default
  if (lastVisibleMenuItem === -1) {
    // render all of them here, same as before
    return ...
  }

  // show "more" button if the last visible item is not the last one in the array
  const isMoreVisible = lastVisibleMenuItem < items.length - 1;

  // filter out those items which index is more than the last visible
  const filteredItems = items.filter((item, index) => index <= lastVisibleMenuItem);

  return (
    <div className="navigation">
      <!-- render only visible items -->
      {filteredItems.map(item => <a href={item.href}>{item.name}</a>)}

      <!-- render "more" conditionally -->
      {isMoreVisible && <button id="more">...</button>}
    </div>
  )
}
```

다만 CPU 성능이 안 좋거나 네트워크가 원활하지 않은 환경에서는 아래와 같은 nav 메뉴들 전체가 순간적으로 나타나는 flickering 현상이 두드러지게 보이는 현상이 발목을 잡게 됩니다🤬

![Slow Network](/images/slow-network.png)

## Fixing it with useLayoutEffect

위와 같은 flickering 문제는 `useEffect` 대신 `useLayoutEffect`를 적용하면 간단히 해결할 수 있습니다.

다만 이 방법은 전반적인 페이지의 성능을 떨어뜨린다는 이유로 [React 공식문서](https://react.dev/reference/react/useLayoutEffect)에서 사용을 권장하지 않습니다.

## Why the fix works: rendering, painting and browsers

브라우저 repaint은 그림판에 낙서를 하듯이 실시간 + 연속으로 이루어지지 않습니다.

대신 아래 flip book과 같이 짧은 시간동안 여러 장의 이미지(=frame)들을 매우 빠르게 넘기면서 UI에 변화를 부여합니다.

![Flip Book](/images/flip-book.gif)

여기서 하나의 frame이 viewport에 그려지는 연산은 단일 task로서 queue에 대기했다가 브라우저에 의해서 동기적으로 처리됩니다.

브라우저에서 동작하는 event loop와 queue에 대해서 더 알고 싶다면 [이 아티클](https://blog.xnim.me/event-loop-and-render-queue)을 참고하시면 됩니다.

만일 브라우저가 느리게 동작하는 등의 이유로 현재 task를 처리하는데 시간이 걸려서 다음 task로 넘기기까지 지연이 발생한다면 움직임이 끊겨서 보이는데 이 현상을 "blocking painting"이라고 합니다.

React는 이와 같은 지연이 발생하지 않도록 거대한 task를 더 작은 task 단위로 분리하여 비동기적으로 처리할 수 있도록 도와줍니다.

### Back to useEffect vs useLayoutEffect

`useLayoutEffect`는 브라우저 repaint 직전에 **동기적으로** 실행되는 effect(layout effect)를 정의합니다.

`useEffect` hook으로 정의한 effect와 다르게 컴포넌트 렌더링과 동일한 task로 간주되기 때문에 만일 layout effect 내에서 컴포넌트 state를 업데이트한다면 즉시 리렌더링을 마친 뒤에 repaint가 수행됩니다.

![useEffect](/images/use-effect.png)

![useLayoutEffect](/images/use-layout-effect.png)

layout effect를 처리하는데 시간이 걸린다면 컴포넌트를 포함한 전체 페이지의 repaint가 지연되고 그동안 사용자는 빈 화면을 빈 화면을 몇 초간 응시할 수도 있습니다.😵

따라서 `useLayoutEffect` hook은 위 예시처럼 DOM measurement를 기반으로 UI 업데이트가 필요한 상황에 한해서만 사용할 필요가 있습니다.

### A bit more about useEffect

`useEffect` hook으로 정의한 effect는 보통 브라우저 repaint 이후에 실행됨을 보장하지만 간혹 특정 상황에 한해서 `useLayoutEffect`처럼 repaint 전에 실행될 수도 있습니다.

자세한 내용은 [이 아티클](https://thoughtspile.github.io/2021/11/15/unintentional-layout-effect/)을 참고하시면 됩니다.

## useLayoutEffect in Next.js and other SSR frameworks

페이지를 SSR 방식에 따라 렌더링(=React 컴포넌트 함수들을 실행하여 얻은 HTML 생성)하는 과정에서 `useLayoutEffect` hook은 실행되지 않습니다.

그리고 서버에서 초기 상태값을 기반으로 생성된 HTML에 모든 nav 메뉴들이 그려지기 때문에 위에서 구현한 동적인 nav 메뉴에 또 다시 flickering 현상이 나타납니다😇

현상을 해결하는 방법은 상황에 따라 달라지겠지만 컴포넌트가 mount되기 전후로 다른 UI가 보이도록 분기처리할 수 있습니다.

```jsx
const Component = () => {
  const [shouldRender, setShouldRender] = useState(false);

  useEffect(() => {
    setShouldRender(true); // 브라우저에서 실행
  }, []);

  if (!shouldRender) return <SomeNavigationSubstitude />; // 서버에서 실행

  return <Navigation />;
};
```

간혹 서버에서 컴포넌트가 렌더링되는지 여부를 기준으로 분기처리를 하는 경우가 있습니다.

```jsx
const Component = () => {
  // Detectign SSR by checking whether window is there
  if (typeof window === undefined) return <SomeNavigationSubstitude />;

  return <Navigation />;
};
```

하지만 해당 로직은 자칫하면 서버에서 내려온 초기 HTML의 내용과 브라우저에서 처음으로 렌더링한 내용에 차이가 생겨서 에러가 발생할 수 있으므로 지양해야 합니다.

hydration 관련해서 유의할 점들은 [이 아티클](https://www.joshwcomeau.com/react/the-perils-of-rehydration/)을 참고하시면 됩니다.
