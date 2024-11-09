---
title: How to useMemo and useCallback, you can remove most of them
date: 2024-11-04 21:21:49
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
`useMemo` 나 `useCallaback` 으로 wrapping된 값이나 함수는 초기 렌더링 때 캐시되고 이후 렌더링에서는 캐싱된 데이터를 참조합니다.

어떻게 보면 컴포넌트의 모든 변수와 함수들을 memoise하는게 좋겠지만 캐싱 연산에 의해서 초기 렌더링에 지연이 발생할 수 있어서 필요할 때만 사용하는 것을 권장합니다.

컴포넌트의 props를 위 두 hook들로 memoise하려면 해당 props를 가지는 컴포넌트도 `React.memo` 로 memoise되어 있어야 합니다.

```jsx
const PageMemoized = React.memo(Page);

const App = () => {
  const [state, setState] = useState(1);
  const onClick = useCallback(() => {
    console.log("Do something on click");
  }, []);

  return (
    // page WILL re-render because value is not memoized
    <PageMemoized onClick={onClick} value={[1, 2, 3]} />
  );
};
```
memoise 대상은 복잡한 JS 연산결과가 아닌 컴포넌트 자체여야 하는데, 컴포넌트 렌더링 연산이 훨씬 더 많은 리소스를 필요로 하기 때문입니다.
