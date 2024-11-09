---
title: How to fetch data in React with performance in mind
date: 2024-11-04 21:16:15
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
data fetching 방식은 2가지가 있습니다.

1. on-demand fetching

    - 사용자와 페이지 사이의 상호작용으로 필요할 때마다 fetch하는 방식
    - React에서는 콜백으로 구현

2. initial fetching

    - 컴포넌트가 렌더링되는 중에 fetch하는 방식
    - React에서는 `useEffect` 로 구현

`useEffect` 로 fetch를 구현했을 때 발생하는 이슈들을 정리한 글인데 한번 참고하시길!

`useEffect` 자체 문제가 아닌 JS에서 비동기 작업을 처리하는 방식에 의해서 발생하는 현상들임을 강조합니다.

[r/reactjs - Comment by u/gaearon on ”What is the recommended way to load data for React 18?”](https://www.reddit.com/r/reactjs/comments/vi6q6f/comment/iddrjue)

data fetch 기능을 개발할 때는 3가지 사항들을 고려해야 합니다.

- 언제 fetch를 시작할 것인가?
- fetch가 진행되는 동안 무엇을 할 것인가?
- fetch가 끝난 뒤에는 무엇을 할 것인가?

컴포넌트의 life cycle은 컴포넌트로부터 반환된 ReactElement가 부모 컴포넌트의 return문에 의해서 반환되는 경우에 시작됩니다.

```jsx
const Parent = () => {
  // set loading to true initially
  const [isLoading, setIsLoading] = useState(true);

  // child is now here! before return
  const child = <Child />;

  if (isLoading) return "loading";

  return child; // 이 지점이 실행되어야 Child 컴포넌트의 life cycle 시작됩니다
};
```

fetch가 일어나는 동안 loader만을 보여주고 fetch가 완료되면 하위 컴포넌트를 렌더링하는 방식으로 구현하면 Request Waterfall 현상이 일어납니다.

먼저 fetch를 수행하는 hook을 아래와 같이 정의합니다.

```jsx
export const useData = (url) => {
  const [state, setState] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await fetch(url).then((data) => data.json());
      setState(data);
    };

    dataFetch();
  }, [url]);

  return { data: state };
};
```

그리고 아래와 같은 구조를 가진 App을 구성하고 컴포넌트마다 fetch가 일어나도록 useData hook을 적용합니다.

```jsx
const App = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData("/get-sidebar");

  // show loading state while waiting for the data
  if (!data) return "loading";

  return (
    <>
      <Sidebar data={data} />
      <Issue />
    </>
  );
};

const Sidebar = ({ data }) => {
  // some sidebar links
  return;
};

const Issue = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData("/get-issue");

  // show loading state while waiting for the data
  if (!data) return "loading";

  // render actual issue now that the data is here!
  return (
    <div>
      <h3>{data.title}</h3>
      <p>{data.description}</p>
      <Comments />
    </div>
  );
};

const Comments = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData("/get-comments");

  // show loading state while waiting for the data
  if (!data) return "loading";

  // rendering comments now that we have access to them!
  return data.map((comment) => <div>{comment.title}</div>);
};
```

이제 App을 렌더링하면 아래 그림과 같이 상위 컴포넌트의 fetch가 종료되어야만 하위 컴포넌트의 life cycle이 실행되어 차례대로 fetch가 일어납니다.

![Classic Waterfall Example.png](/images/classic_waterfall_example.png)

위와 같이 구현된 경우, fetch가 많은 앱이라면 전체 페이지의 컨텐츠가 모두 표시되는 시간이 꽤 걸리게 됩니다🥲

최대한 동시간대에 동시에 fetch를 진행하는 방법들을 정리하면 다음과 같습니다.

## Promise.all

`Promise.all` 메서드를 사용하여 다수의 fetch들을 동시에 시작하여 마지막 fetch까지 정상적으로 완료되었을 때 렌더링을 진행합니다.

```jsx
const useAllData = () => {
  const [sidebar, setSidebar] = useState();
  const [comments, setComments] = useState();
  const [issue, setIssue] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      // waiting for allthethings in parallel
      const result = (
        await Promise.all([
          fetch(sidebarUrl),
          fetch(issueUrl),
          fetch(commentsUrl),
        ])
      ).map((r) => r.json());

      // and waiting a bit more - fetch API is cumbersome
      const [sidebarResult, issueResult, commentsResult] = await Promise.all(
        result
      );

      // when the data is ready, save it to state
      setSidebar(sidebarResult);
      setIssue(issueResult);
      setComments(commentsResult);
    };

    dataFetch();
  }, []);

  return { sidebar, comments, issue };
};
```

```jsx
const App = () => {
  // all the fetches were triggered in parallel
  const { sidebar, comments, issue } = useAllData();

  // show loading state while waiting for all the data
  if (!sidebar || !comments || !issue) return "loading";

  // render the actual app here and pass data from state to children
  return (
    <>
      <Sidebar data={state.sidebar} />
      <Issue comments={state.comments} issue={state.issue} />
    </>
  );
};
```

![Promise All Example.png](/images/promise_all_example.png)

## Parallel promises

이전 방법은 가장 오래 걸리는 fetch에 의해서 다른 fetch들이 완료되어도 계속 대기해야 하는 단점이 있습니다. 그래서 동시에 fetch를 시작하되 개별 fetch가 끝나는대로 개별 로직을 수행하도록 수정하면 됩니다.

```jsx
const useAllData = () => {
  const [sidebar, setSidebar] = useState();
  const [comments, setComments] = useState();
  const [issue, setIssue] = useState();

  useEffect(() => {
    const dataFetch = () => {
      fetch("/get-sidebar")
        .then((data) => data.json())
        .then(setSidebar);

      fetch("/get-issue")
        .then((data) => data.json())
        .then(setIssue);

      fetch("/get-comments")
        .then((data) => data.json())
        .then(setComments);
    };

    dataFetch();
  }, []);

  return { sidebar, comments, issue };
};
```

```jsx
const App = () => {
  const { sidebar, issue, comments } = useAllData();

  // show loading state while waiting for sidebar
  if (!sidebar) return 'loading';

  // render sidebar as soon as its data is available
  // but show loading state instead of issue and comments while we're waiting for them
  return (
    <>
      <Sidebar data={sidebar} />
      <!-- render local loading state for issue here if its data not available -->
      <!-- inside Issue component we'd have to render 'loading' for empty comments as well -->
      {issue ? <Issue comments={comments} issue={issue} /> : 'loading''}
    </>
  )
```

![Parallel Promises Example.png](/images/parallel_promises_example.png)

하지만 이 방식과 이전 방식의 문제점은 fetch가 끝나는 대로 state를 수정하기 때문에 총 3번의 App 전체 리렌더링이 일어난다는 겁니다.

## Data providers to abstract away fetching

컴포넌트의 상태를 상위 컴포넌트로 lift up하면 props drilling에 의해서 가독성이 떨어질 수 있습니다. 그래서 fetch하여 상태를 업데이트하는 로직을 개별 context에 구현하여 필요한 부분만 리렌더링하도록 구현할 수 있습니다.

```jsx
const Context = React.createContext();

export const CommentsDataProvider = ({ children }) => {
  const [comments, setComments] = useState();

  useEffect(async () => {
    fetch("/get-comments")
      .then((data) => data.json())
      .then(setComments);
  }, []);

  return <Context.Provider value={comments}>{children}</Context.Provider>;
};

export const useComments = () => useContext(Context);
```

```jsx
const App = () => {
  const sidebar = useSidebar();
  const issue = useIssue();

  // show loading state while waiting for sidebar
  if (!sidebar) return 'loading';

  // no more props drilling for any of those
  return (
    <>
      <Sidebar />
      {issue ? <Issue /> : 'loading''}
    </>
  )
}
```

그리고 App을 아래와 같이 개별 provider들로 wrapping하면 됩니다.

```jsx
export const VeryRootApp = () => {
  return (
    <SidebarDataProvider>
      <IssueDataProvider>
        <CommentsDataProvider>
          <App />
        </CommentsDataProvider>
      </IssueDataProvider>
    </SidebarDataProvider>
  );
};
```

## What if I fetch data before React?

생각해보면 굳이 fetch하는 부분을 dependency가 없는 `useEffect`에 구현하는 것보다 컴포넌트 파일을 load하자마자 promise를 생성하는 것도 하나의 방법이 아닐까요?

```jsx
const commentsPromise = fetch("/get-comments");

const Comments = () => {
  useEffect(() => {
    const dataFetch = async () => {
      // just await the variable here
      const data = await commentsPromise.then((data) => data.json());

      setState(data);
    };

    dataFetch();
  }, [url]);
};
```

위와 같이 구현하면 `useEffect`가 실행되는 시점보다 더 일찍 fetch를 수행할 수 있습니다.

하지만 아래와 같이 fetch의 호출시점과 완료시점을 제어할 수 없을 뿐만 아니라 브라우저의 동시 요청개수가 모르는 사이에 초과되어 중요한 영역의 fetching과 렌더링을 지연시킬 수 있는 잠재적 위험성이 있습니다.

![Fetch Outside.png](/images/fetch_outside.png)

그래서 위와 같은 fetch 방식은 route 이동 중에 pre-fetch하거나 lazy-loading되는 컴포넌트에서 사용할 data를 pre-fetching하는데 사용할 때만 적합합니다.


현대 브라우저들은 동일한 host에 대해서 병렬적으로 처리할 수 있는 요청들의 개수가 제한되어 있습니다(Chrome의 경우 6개).

만일 그보다 더 많은 요청이 생성되면 queue에 저장되어 순서대로 처리됩니다.
