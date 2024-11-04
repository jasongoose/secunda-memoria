---
title: React useTransition, performance game changer or...?
date: 2024-11-04 21:20:52
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
## Let's implement a slow state update

아래와 같이 Tab을 구현한 컴포넌트가 있습니다.

```jsx
export default function App() {
  const [tab, setTab] = useState("issues");

  return (
    <div className="container">
      <div className="tabs">
        <TabButton onClick={() => setTab("issues")} name="Issues" />
        <TabButton onClick={() => setTab("projects")} name="Projects" />
        <TabButton onClick={() => setTab("reports")} name="Reports" />
      </div>
      <div className="content">
        {tab === "issues" && <Issues />}
        {tab === "projects" && <Projects />}
        {tab === "reports" && <Reports />}
      </div>
    </div>
  );
}
```

위 예시에서 `tab` state가 `projects`로 업데이트된다면 리렌더링에 의해서 `Projects` 컴포넌트가 그려질겁니다.

여기서 `Projects` 컴포넌트를 렌더링하는데 시간이 걸려서 렌더링 중간에 `tab`을 `issues`로 변경한다면 예상과 다르게 `Issues` 컴포넌트가 바로 그려지지 않고 일시적으로 UI에 반응이 없는 현상이 일어나는데요.

이 부분은 React가 컴포넌트의 리렌더링(trigger -> render -> commit)을 동기적으로 처리하기 때문에 발생합니다.

브라우저는 일련의 state update들을 batch로 처리하는 리렌더링 작업을 task 단위로 queue에서 관리하고 실행합니다.

그래서 먼저 enqueue된 task가 끝날 때까지는 이후에 들어온 다른 task가 실행되지 않고 대기하면서 지연이 발생하는 겁니다.

## Concurrent Rendering and useTransition for slow state updates

React v18에서는 위와 같은 Render Blocking 현상을 해결하기 위해 Concurrent Rendering이란 개념을 도입합니다.

해당 렌더링 방식은 리렌더링 task에 우선순위를 부여하여 임의의 task를 처리하는 중에 우선순위가 높은 task가 들어오면 기존 task를 중지하고 더 급한 task를 먼저 처리한 다음에 중지되었던 task를 다시 처리하거나 버리고 새로운 task를 시작하는 방식을 가지는데요.

브라우저 런타임은 task를 우선순위별 queue에서 관리하고 event loop에 기반한 context switch로 실행할 task를 바꿀 수 있습니다.

React의 concurrency는 `useTransition` hook 또는 `useDeferredValue` hook으로 선택적으로 구현할 수 있습니다.

위 예시에서 blocking 현상을 일으키는 state update를 `useTransition` hook에서 반환된 `startTransition` 함수의 인자로 전달하면 해당 state update로 실행되는 rendering task의 우선순위가 낮춰서 다른 task에 의해서 전환(transit, switch)되도록 만듭니다(=interruptable).

여기서 우선순위가 낮은 task를 background task라고도 불리는데 당연한 소리지만 commit 되기 전까지는 화면에 반영되지 않습니다.

그리고 `isPending`은 background task의 처리상태를 나타내는 상태값으로 임의의 리렌더링이 완료되었는지 여부를 확인할 수 있습니다.

```jsx
export default function App() {
  const [tab, setTab] = useState("issues");

  // add the useTransition hook
  const [isPending, startTransition] = useTransition();

  return (
    <div className="container">
      <div className="tabs">
        ...
        <TabButton
          // indicate that the content is loading
          isLoading={isPending}
          onClick={() => {
            // call setTab inside a function
            // that is passed to startTransition
            startTransition(() => {
              setTab("projects");
            });
          }}
          name="Projects"
        />
        ...
      </div>
      ...
    </div>
  );
}
```

그럼 `tab`의 값이 `issues` -> `projects` -> `reports` 순으로 변경되면 `reports` 기준으로 리렌더링하는 task가 우선되면서 원래 있던 blocking을 해결할 수 있습니다.

## The dark side of useTransition and re-renders

`useTransition`으로 blocking을 방지할 수 있다는 이유로 아래와 같이 모든 `tab`의 state update를 `startTransition`으로 감싼다면 오히려 뜻밖의 결과가 나올 수 있습니다.

```jsx
const onTabClick = (tab) => {
  startTransition(() => {
    setTab(tab);
  });
};
```

사실 background task를 실행한다면 [2번에 걸쳐서 리렌더링이 일어납니다.](https://github.com/facebook/react/issues/24269#issuecomment-1086967895)

먼저 기존 state를 기준으로 즉시 렌더링(="urgent render")이 일어나고 나중에 들어온 task에 대해서 `isPending`의 값이 `false`에서 `true`로 변경됩니다.

즉시 렌더링이 끝나고 나서야 pending 상태에 있던 다음 task(="concurrent render")가 처리되면서 새로운 state를 기준으로 렌더링이 다시 한번 일어납니다.

## How to use useTransition, then?

### Memoization to everything

위와 같이 `useTransition`을 사용하면서 초기 리렌더링에 의한 blocking을 막으려면 memoization으로 렌더링 관련 연산량을 줄일 필요가 있습니다.

이를 위해서 아래와 같이 캐싱을 적용할 수 있습니다.

- 무거운 컴포넌트에 `React.memo`를, props로 전달되는 객체나 함수에 `useMemo`나 `useCallback`으로 처리
- 무거운 연산은 `React.memo`로 처리

여기서 `isPending`이 `useMemo`, `useCallback`의 dependency 또는 props로 사용되어서는 절대 안됩니다!

```jsx
const IssuesMemo = React.memo(Issues);
const ProjectsMemo = React.memo(Projects);
const ReportsMemo = React.memo(Reports);
```

하지만 아래와 같이 memoization을 잘못 적용할 경우를 유의해야 합니다.

```jsx
const ListMemo = React.memo(List);
// -> const ListMemo = useMemo(() => <ListMemo /> , []);
const IssuesMemo = React.memo(Issues);

const App = () => {
  // if startTransition is triggered, will IssuesMemo re-render? -> YES!
  const [isPending, startTransition] = useTransition();

  return (
    ...
    <IssuesMemo>
      <ListMemo />
    </IssuesMemo>
  )
}
```

`children` props로 전달된 `ListMemo` 컴포넌트는 `App`이 리렌더링될 때마다 새로 생성되므로 `IssuesMemo` 컴포넌트 또한 리렌더링 됩니다.

### Transition from nothing to heavy

캐싱 처리 외에도 data fetching과 같이 그저 무에서 유를 만드는 state update에 한해서 `useTransition`을 적용하는 방법도 있습니다.

```jsx
const App = () => {
  const [data, setData] = useState();

  useEffect(() => {
    fetch('/some-url').then((result) => {
      // lots of data
      startTransition(() => {
      	setData(result);
      });
    })
  }, [])

  if (!data) return 'loading'

  return ... // render that lots of data when available
}
```

Effect에 있는 `setData`에 `startTransition`을 적용한다면 초기 렌더링으로 `data`가 `undefined`가 되면서 "loading"이 표시되기 때문에 blocking이 발생하지 않습니다.

## What about useDeferredValue?

`useDeferredValue`도 `useTransition`처럼 concurrency를 구현할 때 사용하는 함수입니다.

다만 state update 함수를 사용할 수 없는 컴포넌트 단에서 props나 state의 변화에 의한 리렌더링 task의 우선순위를 낮춘다는 점에서 동작방식이 다릅니다.

```jsx
const TabContent = ({ tab }) => {
  // mark the "tab" value as non-critical
  const tabDeffered = useDeferredValue(tab);

  return (
    <>
      {tabDeffered === "issues" && <Issues />}
      {tabDeffered === "projects" && <Projects />}
      {tabDeffered === "reports" && <Reports />}
    </>
  );
};
```

물론 위와 같이 구현하면 초기 리렌더링에 의한 blocking이 발생하기 때문에 적절한 memoization이 똑같이 필요합니다.

## Can I use useTransition for debouncing?

아래와 같이 키보드로 입력한 값을 `value` state로 업데이트하는 로직을 `startTransition`을 적용해도 키보드 하나씩 입력하는 사이에 React가 background task를 commit까지 처리하여 debounce 효과가 나타나지 않습니다.

```jsx
function App() {
  const [value, setValue] = useState("");
  const [isPending, startTransition] = useTransition();

  const onChange = (e) => {
    startTransition(() => {
      setValue(e.target.value);
    });
  };

  // Effect는 렌더링 task가 commit까지 완료해서 화면에 반영된 이후에 실행됩니다~
  useEffect(() => {
    console.log("Value: ", value);
  }, [value]);

  return (
    <>
      <input type="text" onChange={onChange} />
    </>
  );
}
```

대신 `useDeferredValue` hook을 적용하면 background task가 commit되기 전까지 이전 state를 참조할 수 있어서 debounce 효과를 낼 수 있습니다.

```jsx
function App() {
  const [value, setValue] = useState("");
  const deferredValue = useDeferredValue(value);

  const onChange = (e) => {
    setValue(e.target.value);
  };

  return (
    <>
      <input type="text" onChange={onChange} />
    </>
  );
}
```

최근에 일어난 렌더링으로 `deferredValue`는 `value`와 동일한 값을 가지고 있습니다.

위 예시에서 `value`의 업데이트로 `App`의 리렌더링이 일어나면 그 과정에서 `useDeferredValue` hook을 이전 렌더링 때와 다른 값으로 호출하면서 2번의 리렌더링이 일어납니다.

업데이트된 `value`와 이전 `value`의 값을 가진 `deferredValue`를 기준으로 즉시 렌더링이 일어나고, 업데이트 `value`와 동일한 값을 가진 `deferredValue`를 기준으로 background task가 처리됩니다.

그래서 키보드로 입력할 때마다 background task가 중간에 계속 내쳐지지만 `deferredValue`는 최근에 commit된 값을 그대로 가집니다.
