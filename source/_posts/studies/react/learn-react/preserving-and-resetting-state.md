---
title: Preserving and Resetting State
date: 2024-11-04 08:13:17
categories:
  - Studies
  - React
  - Learn React
#tags:
---
컴포넌트 내부에서 정의한 state는 사실 컴포넌트 함수 context가 아닌 React라는 컴포넌트 외부 영역(객체)에서 컴포넌트별로 관리됩니다.

React는 컴포넌트가 상위 컴포넌트의 동일한 위치에 표시되는 동안에는 state를 유지하고, 제거되거나 다른 타입의 컴포넌트가 해당 위치에서 렌더링한다면 이전 컴포넌트의 state를 제거합니다.

위치를 파악할 때 JSX 내의 expression(`{…}`)을 하나의 Node로 간주합니다.

```jsx
<div>
  {showB && <Counter />} // showB가 false여도
  <Counter /> // 이 컴포넌트의 위치는 변하지 않는 것으로 간주합니다.
  <label>
    <input
      type="checkbox"
      checked={showB}
      onChange={(e) => {
        setShowB(e.target.checked);
      }}
    />
    Render the second counter
  </label>
</div>
```

**따라서 리렌더링 과정에서 특정 컴포넌트의 state를 유지하려면 렌더링 전과 후에 해당 컴포넌트의 상위 컴포넌트 상의 위치가 동일하도록 구조를 수정하면 됩니다.**

하지만 개발하다보면 동일한 타입의 컴포넌트를 동일한 위치에 조건부로 여러 번 사용하지만 별개의 state를 가지는 경우가 필요할 때가 있습니다.

```jsx
<div>
  {isPlayerA ? (
    <Counter person="Taylor" /> // 0, 1, 2, _, _, _, ...
  ) : (
    <Counter person="Sarah" /> //  _, _, _, 3, 4, 5, ...
  )}
  <button
    onClick={() => {
      setIsPlayerA(!isPlayerA);
    }}
  >
    Next player!
  </button>
</div>
```

이 경우, 컴포넌트에 `key` attribute를 지정하면 React로 하여금 동일한 컴포넌트라도 각 상태를 다르게 처리합니다.

```jsx
<div>
  {isPlayerA ? (
    <Counter key="Taylor" person="Taylor" /> // 0, 1, 2, _, _, _, ...
  ) : (
    <Counter key="Sarah" person="Sarah" /> //   _, _, _, 0, 1, 2, ...
  )}
  <button
    onClick={() => {
      setIsPlayerA(!isPlayerA);
    }}
  >
    Next player!
  </button>
</div>
```

`key` attribute은 상위 컴포넌트 내부에서의 컴포넌트의 위치 정보이자 동일한 컴포넌트들 간의 내부 state를 구분하기 위한 id 역할을 합니다.
