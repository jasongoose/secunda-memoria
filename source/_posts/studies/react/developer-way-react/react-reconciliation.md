---
title: React reconciliation, how it works and why should we care
date: 2024-11-04 21:20:35
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
## React reconciliation algorithm

특정 컴포넌트에 `<input />`을 표시하려면 아래와 같이 컴포넌트를 정의할 수 있습니다.

```jsx
const Input = ({ placeholder }) => {
  return <input type="text" />;
};

// somewhere else
<Input placeholder="Input something here" />;
```

만일 placeholder를 업데이트하고 싶다면 기존 `<input />`을 unmount하고 새로운 `<input />`으로 mount할 수도 있습니다.

하지만 리마운트(re-mount)하는 대신 아래와 같은 스크립트로 대상 `<input />`을 지정하여 placeholder만을 바꾸는 것이 더 빠를 겁니다.

```js
const input = document.getElementById("input-id");
input.placeholder = "new data";
```

React는 실제 DOM을 모방하여 화면에 렌더링되는 element들의 정보를 트리 구조의 단일 객체로 정의하는데 이를 VDOM이라고 합니다.

Reconciliation은 리렌더링 전후의 VDOM 사이의 변경사항(diff)를 계산하는 과정으로 DOM node를 추가, 삭제, 정렬하는 등의 DOM manipulation 연산을 최소화하기 위해서 필요합니다.

함수로 정의한 컴포넌트로부터 반환된 JSX 구문은 Babel에 의해서 `type`, `props` 등의 속성을 가진 객체 즉, React Element(이하 react-el)로 변환됩니다.

```jsx
// as-is
const Input = () => {
  return <input type="text" />;
};

// to-be
{
  type: "input", // type of element that we need to render
  props: {...}, // input's props like id or placeholder
  ... // bunch of other internal stuff
}
```

만일 Fragment 사용으로 다수 root를 가지는 컴포넌트라면 배열 형태로 변환됩니다.

```jsx
// as-is
const Input = ({ label }) => {
  return (
    <>
      <label htmlFor="id">{ label }</label>
      <input type="text" id="id" />
    </>
  );
};

// to-be
[
  {
    type: 'label',
    ... // other stuff
  },
  {
    type: 'input',
    ... // other stuff
  }
]
```

`<input />`, `<label />`과 같이 HTML DOM element에 해당하는 react-el은 element 이름이 `type`으로 지정됩니다.

이와 다르게 React 컴포넌트는 렌더링 context 상에 선언된 컴포넌트 함수 자체가 `type`으로 지정됩니다.

```jsx
const Component = () => {
  return <Input />;
};

{
  type: Input, // reference to that Input function we declared earlier
  ... // other stuff
}
```

컴포넌트 최초 mount 시 React는 개별 react-el 객체의 `type`의 타입에 맞춰서 DOM node를 생성하고 `appendChild` 메서드로 부모 node에 append합니다.

- `type`이 string 타입 -> HTML element 생성
- `type`이 함수 타입 -> 함수를 실행하여 반환된 VDOM을 순회하여 동일한 연산을 재귀적으로 수행

```jsx
// 컴포넌트 함수
const Component = () => {
  return (
    <div>
      <Input placeholder="Text1" id="1" />
      <Input placeholder="Text2" id="2" />
    </div>
  );
};

// 생성된 react-el
{
  type: 'div',
  props: {
    // children are props!
    children: [
      {
        type: Input,
        props: { id: "1", placeholder: "Text1" }
      },
      {
        type: Input,
        props: { id: "2", placeholder: "Text2" }
      }
    ]
  }
}

// 최종 DOM node
<div>
  <input placeholder="Text1" id="1" />
  <input placeholder="Text2" id="2" />
</div>
```

## Reconciliation and state update

VDOM 상에 특정 컴포넌트의 state가 업데이트되면 root로부터 리렌더링이 일어납니다.

React는 컴포넌트에서 리렌더링 전후로 반환된 react-el의 `type` 속성 변화 여부에 따라 다르게 처리합니다.

- 변화가 있으면 컴포넌트가 호출된 위치에 있는 이전 react-el을 unmount하고 이후 react-el을 mount합니다.

- 변화가 없다면 **기존 react-el의 state와 DOM을 그대로 유지한 채로(=기존 컴포넌트를 재사용하여)** 리렌더링합니다.

컴포넌트의 state의 결과는 리렌더링 과정에서 유지되거나 unmount에 의해서 없어지는 경우로 나뉩니다!

```jsx
const Component = () => {
  if (isCompany) return <Input />;

  return <TextPlaceholder />;
};
```

위 예시의 경우 `isCompany` state가 토글링되면서 아래와 같이 react-el의 타입이 달라지므로 리마운트가 일어납니다.

```jsx
// Before update, isCompany was "true"
{
  type: Input,
  ...
}

// After update, isCompany is "false"
{
  type: TextPlaceholder,
  ...
}
```

하지만 아래와 같이 동일한 `type`을 가진다면(=동일한 `Input` 함수를 참조한다면) 달라진 props를 기준으로 리렌더링이 일어납니다.

```jsx
const Form = () => {
  const [isCompany, setIsCompany] = useState(false);

  return (
    <>
      <Checkbox onChange={() => setIsCompany(!isCompany)} />
      {isCompany ? (
        <Input id="company-tax-id-number" placeholder="Enter you company Tax ID" ... />
      ) : (
        <Input id="person-tax-id-number" placeholder="Enter you personal Tax ID" ... />
      )}
    </>
  )
}
```

## Reconciliation and arrays

리렌더링 전후로 동일한 타입의 컴포넌트를 사용하되 리마운트가 일어나도록 만들려면 배열 구조의 react-el을 활용할 수 있습니다.

예를 들어 직전 예시를 아래와 같이 수정할 수 있습니다.

```jsx
const Form = () => {
  const [isCompany, setIsCompany] = useState(false);

  return (
    <>
      ... // checkbox somewhere here
      {isCompany ? <Input id="company-tax-id-number" ... /> : null}
      {!isCompany ? <Input id="person-tax-id-number" ... /> : null}
    </>
  )
}
```

그럼 `Form`에서 반화되는 react-el은 `isCompany`의 값에 따라 아래와 같이 달라집니다.

```jsx
// isCompany === false
[{ type: Checkbox }, null, { type: Input }][
  // isCompany === true
  ({ type: Checkbox }, { type: Input }, null)
];
```

여기서 React는 배열 구조의 react-el의 diff를 계산할 때 `type`과 배열 상의 위치인 index도 함께 고려합니다.

위에서 index_0에서 `Checkbox`는 그대로 유지되지만 index_1, index_2에서는 `null` 또는 `Input`으로 `type`이 변하므로 해당 위치에서는 리마운트가 일어납니다.

## Reconciliation and "key"

리렌더링 전후로 동일한 타입의 컴포넌트를 사용하되 리마운트가 일어나도록 만드는 2번째 방법으로는 `key`를 사용하는 겁니다.

컴포넌트의 props처럼 지정되는 `key`는 배열 구조의 react-el를 구성하는 child element들을 구분하기 위한 고유 id로서 index 대신 활용됩니다.

조건에 따라 렌더링되는 item의 개수, 종류, 순서 등이 달라지는 dynamic array에서 `key`의 역할은 중요하기 때문에 필수로 지정되어야만 합니다.

만일 아래와 같은 리스트 렌더링에서 index가 `key`로 동작한다면 [리렌더링 전후로 구성하는 react-el의 index는 그대로 동일하기 때문에 개별 react-el이 달라진 state를 반영하지 않는 결과가 나올 수도 있습니다.](https://velog.io/@jasongoose/React-key-attribute-best-practices-for-performant-lists)

```jsx
// as-is
[
  { type: Input }, // key=0 | "1" data item,
  { type: Input }, // key=1 | "2" data item,
];

// to-be
[
  { type: Input }, // key=0 | "2" data item now, but React doesn't know that,
  { type: Input }, // key=1 | "1" data item now, but React doesn't know that,
];
```

그래서 `key` 값으로 구성 react-el이 표시하는 데이터를 표현하면서 고유한 id로 지정한다면 React는 동일한 `type`과 `key`를 가진 react-el의 state와 DOM을 그대로 재사용할 수 있고 이를 DOM에 반영할 수 있습니다.

```jsx
// as-is
[
  { type: Input, key: "1" }, // "1" data item
  { type: Input, key: "2" }, // "2" data item
];

// to-be
[
  { type: Input, key: "2" }, // "2" data item, React knows that because of "key"
  { type: Input, key: "1" }, // "1" data item, React knows that because of "key"
];
```

## Using "key" to force reuse of an existing element

위 내용과 반대로 배열 구조의 react-el을 구성하는 child element에 동일한 `key`를 부여한다면 리렌더링 전에 사용하던 state와 DOM을 그대로 재사용할 수 있습니다.

```jsx
<>
  <Checkbox onChange={() => setIsCompany(!isCompany)} />
  {isCompany ? <Input id="company-tax-id-number" key="tax-input" ... /> : null}
  {!isCompany ? <Input id="person-tax-id-number" key="tax-input" ... /> : null}
</>

// as-is
[
  { type: Checkbox },
  null,
  { type: Input, key: 'tax-input' },
];

// to-be
[
  { type: Checkbox },
  { type: Input, key: "tax-input" }
  null
]
```

위 예시에서 두 `Input`은 `id` props는 다르지만 React 입장에서는 동일한 react-el로 처리됩니다.

## Dynamic arrays and normal elements together

아래와 같이 dynamic array와 위치가 고정된 static element를 렌더링하는 경우도 종종 있습니다.

```jsx
const [data, setData] = useState(['1', '2']);

const Component = () => {
  return (
    <>
      {data.map((i) => <Input key={i} id={i} />)}
      <!-- will this input re-mount if I add a new item in the array above? -->
      <Input id="3" />
    </>
  )
}

[
  { type: Input, key: 1 }, // input from the array
  { type: Input, key: 2 }, // input from the array
  { type: Input }, // input after the array
];
```

그럼 `data`의 길이가 달라진다면 가장 마지막에 있는 `Input`의 `key`인 index는 계속 변하고 `Component`가 리렌더링될 때마다 리마운트가 일어날텐데 성능 상 좋지 않은 영향을 줄 수도 있습니다🤮

하지만 다행스럽게도 React는 위와 같은 상황에서 dynamic array와 static element를 구분하여 react-el을 생성하기 때문에 static element의 리마운트를 방지해줍니다🤩

```jsx
[
  // the entire dynamic array is the first position in the children's array
  [
    { type: Input, key: 1 },
    { type: Input, key: 2 },
  ],
  {
    type: Input, // this is our manual Input after the array
  },
];
```

## Why we can't define components inside other components

아래와 같이 컴포넌트 함수 안에 다른 컴포넌트 함수를 정의하여 렌더링하는 방식은 React Anti-pattern들 중 하나입니다.

```jsx
const Component = () => {
  const Input = () => <input />;

  return <Input />;
};
```

위 예시에서 `Component`가 리렌더링될 때마다 `Input`이 참조하는 컴포넌트 함수 즉, react-el의 `type`이 달라지면서 리마운트가 발생하기 때문에 빈번한 DOM manipulation이 일어납니다.

```jsx
{
  type: Input,
}
```

따라서 컴포넌트는 별도의 함수로 구현하는 것이 권장됩니다!

```jsx
const Input = () => <input />;

const Component = () => {
  return <Input />;
};
```
