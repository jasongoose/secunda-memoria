---
title: React components composition, how to get it right
date: 2024-11-04 21:19:58
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
React와 같은 컴포넌트 기반 라이브러리, 프레임워크로 개발하다보면 어느 영역을 컴포넌트로 분리해서 구성할지에 대해서 대부분의 시간을 쓰게됩니다😩

컴포넌트는 크게 아래와 같은 타입으로 나눌 수 있습니다.

## Simple Components

props, state를 가지는 컴포넌트로, 다른 컴포넌트들을 렌더링하여 compose할 수 있습니다.

```jsx
const Button = ({ title, onClick }) => (
  <button onClick={onClick}>{title}</button>
);
```

```jsx
const Navigation = () => {
  return (
    <>
      // Rendering out Button component in Navigation component. Composition!
      <Button title="Create" onClick={onClickHandler} />
      ... // some other navigation code
    </>
  );
};
```

## Container Components

`children` props를 가지는 컴포넌트입니다.

```jsx
// the code is exactly the same! just replace "title" with "children"
const Button = ({ children, onClick }) => {
  return <button onClick={onClick}>{children}</button>;
};
```

```jsx
const Navigation = () => {
  return (
    <>
      <Button onClick={onClickHandler}>Create</Button>
      ... // some other navigation code
    </>
  );
};
```

## 글쓴이는...

개발하면서 아래 4가지 사항의 필요성을 느낀다고 합니다.

- 구현은 언제나 페이지 단위에서 시작한다.
- 정말로 필요할 때만 컴포넌트 단위로 뺀다.
- 언제나 simple component 단위로 시작하고 정말로 필요할 때만 별의별 테크닉을 사용한다.

그래서 이를 기반으로 하는 컴포넌트 규칙들을 정리하면 다음과 같습니다.

- 현재 컴포넌트의 크기가 매우 큰 경우에 하위 컴포넌트 단위로 나눈다.
- 컴포넌트는 개별 기능을 가진 컴포넌트들로 구성한다(1 컴포넌트 = 1 기능).
- Container 컴포넌트는 Layout과 같이 ReactElement(`children` 대상)들을 감싸는 시각적, 기능적인 로직을 재사용할 필요가 있을 때 따로 정의한다.
- 기능과는 연관성이 없는 상태관리 로직으로부터 분리되도록 컴포넌트를 구성한다.

글쓴이가 말하는 좋은 컴포넌트는 다음을 만족한다고 합니다.

- 스크롤 없이 코드를 분석할 수 있다.
- 기능을 제대로 설명하는 이름을 가진다.
- 선언된 목적 외에 다른 기능을 가지지 않는다.
- 구현방식을 쉽게 이해할 수 있다.
