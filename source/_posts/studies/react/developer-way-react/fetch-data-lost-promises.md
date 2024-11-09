---
title: Fetching data in React, the case of lost Promises
date: 2024-11-04 21:16:29
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
대부분 Promise를 사용하여 data fetching을 구현하는데 Promise는 race condition을 일으킬 수 있습니다.

가령 Page 컴포넌트를 아래와 같이 구현해봅시다.

```jsx
const Page = ({ id }: { id: string }) => {
  const [data, setData] = useState({});

  // pass id to fetch relevant data
  const url = `/some-url/${id}`;

  useEffect(() => {
    fetch(url)
      .then((r) => r.json())
      .then((r) => {
        // save data from fetch request to state
        setData(r);
      });
  }, [url]);

  // render data
  return (
    <>
      <h2>{data.title}</h2>
      <p>{data.description}</p>
    </>
  );
};
```

Page는 첫 mount와 리렌더링되면서 id가 변할 때마다 `useEffect` 콜백을 실행합니다.

여기서 fetch 함수는 promise 객체를 반환하기 때문에 main 스레드는 fetch의 작업이 완료될 때까지 기다리지 않고 곧바로 콜백을 종료합니다.

여기서 이전 fetch가 종료되어 setData 함수를 실행하기도 전에 다음 fetch가 진행된다면 아래와 같이 최근 fetch의 결과가 이전 fetch의 결과를 override할 수도 있습니다.

![First Race Condition.png](/images/first_race_condition.png)

아니면 다음 fetch가 이전 fetch에 의해 override될 수도 있습니다.

![Second Race Condition.png](/images/second_race_condition.png)

이러한 race condition은 개발 중 버그의 원인이 될 수 있으므로 방지해야 합니다.

## **Fixing race conditions: force re-mounting**

이전 fetch를 담당한 promise를 무효화하도록 강제로 re-mount합니다.

컴포넌트를 re-mount하면 이전 상태와 `useEffect` 콜백도 무효화되기 때문에 promise가 settled 상태가 되어도 이후 로직을 수행할 수 없습니다.

```jsx
const App = () => {
  const [page, setPage] = useState("issue");

  return (
    <>
      {page === "issue" && <Issue />}
      {page === "about" && <About />}
    </>
  );
};
```

![Fixing Condition Example.png](/images/fixing_condition_example.png)

컴포넌트의 re-mount는 key props로도 간단히 구현할 수 있습니다.

```jsx
<Page id={page} key={page} />
```

빈번한 컴포넌트 re-mount는 앱 성능에 좋지 않고 자손 컴포넌트의 예상치 못한 `useEffect` 가 실행될 수 있어서 권장하지 않는 방법입니다.

## **Fixing race conditions: drop incorrect result**

최근 fetch의 id를 ref에 저장하고 매 fetch마다 fetch의 결과물이 ref의 id에 대응되는 경우에만 상태 업데이트를 진행합니다.

```jsx
const Page = ({ id }) => {
  // create ref
  const ref = useRef(id);

  useEffect(() => {
    // update ref value with the latest id
    ref.current = id;

    fetch(`/some-data-url/${id}`)
      .then((r) => r.json())
      .then((r) => {
        // compare the latest id with the result
        // only update state if the result actually belongs to that id
        if (ref.current === r.id) {
          setData(r);
        }
      });
  }, [id]);
};
```

## **Fixing race conditions: drop all previous results**

`useEffect` 의 cleanup function을 사용하는데, cleanup은 다음 리렌더링의 `useEffect` 콜백이 실행되기 전에 수행됩니다.

```jsx
useEffect(() => {
  // local variable for useEffect's run
  let isActive = true;

  // do fetch here

  return () => {
    // local variable from above
    isActive = false;
  };
}, [url]);
```

위 예시에서 매 리렌더링마다 `useEffect` 의 콜백은 새로 생성되기 때문에 block 내부 isActive의 값은 `true`입니다.

여기서 cleanup 함수는 closure 현상에 의해서 이전 snapshot의 effect에 정의된 isActive 변수에 접근할 수 있습니다.

그럼 아래와 같이 cleanup 함수에서 isActive 값을 `false`로 지정하면 이전 `useEffect` 에 의해서 실행된 fetch가 resolve될 시점에도 `false`이므로 상태 업데이트를 방지할 수 있습니다.

```jsx
useEffect(() => {
  // set this closure to "active"
  let isActive = true;

  fetch(`/some-data-url/${id}`)
    .then((r) => r.json())
    .then((r) => {
      // if the closure is active - update state
      if (isActive) {
        setData(r);
      }
    });

  return () => {
    // set this closure to not active before next re-render
    isActive = false;
  };
}, [id]);
```

## **Fixing race conditions: cancel all previous requests**

위 방식처럼 이전 fetch들을 무효화하는 또 방법으로 [AbortController API](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)를 사용합니다.

```jsx
useEffect(() => {
  // create controller here
  const controller = new AbortController();

  // pass controller as signal to fetch
  fetch(url, { signal: controller.signal })
    .then((r) => r.json())
    .then(setData)
    .catch((error) => {
      // error because of AbortController
      if (error.name === "AbortError") {
        // do nothing
      } else {
        // do something, it's a real error!
      }
    });

  return () => {
    // abort the request here
    controller.abort();
  };
}, [url]);
```

위와 같이 구현하면 컴포넌트가 리렌더링될 때마다 isActive 값을 비교하지 않아도 되면서 개별적으로 에러 핸들링이 가능해집니다.
